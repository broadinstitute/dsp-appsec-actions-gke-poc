# dsp-appsec-sysbox-gke-poc
PoC for Sysbox rootless Docker on GKE

1. Set up a GKE cluster according to https://github.com/nestybox/sysbox/blob/master/docs/user-guide/install-k8s-cloud.md
    * You can assign node pool label `sysbox-install=yes` (not be be confused with cluster label) right at deploy time.

2. Create Sysbox resources:
    ```
    kubectl apply -f https://raw.githubusercontent.com/nestybox/sysbox/master/sysbox-k8s-manifests/rbac/sysbox-deploy-rbac.yaml
    kubectl apply -f https://raw.githubusercontent.com/nestybox/sysbox/master/sysbox-k8s-manifests/daemonset/sysbox-deploy-k8s.yaml
    kubectl apply -f https://raw.githubusercontent.com/nestybox/sysbox/master/sysbox-k8s-manifests/runtime-class/sysbox-runtimeclass.yaml
    ```

3. Deploy a sample Pod:
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: ubu-bio-systemd-docker
      annotations:
        io.kubernetes.cri-o.userns-mode: "auto:size=65536"
    spec:
      runtimeClassName: sysbox-runc
      containers:
      - name: ubu-bio-systemd-docker
        image: registry.nestybox.com/nestybox/ubuntu-bionic-systemd-docker
        command: ["/sbin/init"]
      restartPolicy: Never
    ```

4. Enter the pod and run a sample Docker rootless 🎉 container in it:
    ```
    ❯ kubectl exec ubu-bio-systemd-docker -it bash
    kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
    root@ubu-bio-systemd-docker:/# docker run --rm -it alpine
    Unable to find image 'alpine:latest' locally
    latest: Pulling from library/alpine
    29291e31a76a: Pull complete 
    Digest: sha256:eb3e4e175ba6d212ba1d6e04fc0782916c08e1c9d7b45892e9796141b1d379ae
    Status: Downloaded newer image for alpine:latest
    ```
