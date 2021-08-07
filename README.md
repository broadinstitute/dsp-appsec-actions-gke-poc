# dsp-appsec-sysbox-gke-poc

![status](https://github.com/broadinstitute/dsp-appsec-sysbox-gke-poc/actions/workflows/github-actions-demo.yml/badge.svg)

PoC for Sysbox rootless Docker on GKE

1. Set up a GKE cluster according to https://github.com/nestybox/sysbox/blob/master/docs/user-guide/install-k8s-cloud.md
    * You can assign node pool label `sysbox-install=yes` (not be be confused with cluster label) right at deploy time.
    * Use [Metadata Concealment](https://cloud.google.com/kubernetes-engine/docs/how-to/protecting-cluster-metadata#concealment), [Workload Identity](https://cloud.google.com/kubernetes-engine/docs/how-to/workload-identity) and/or a [least-privilege Service Account for the nodes](https://cloud.google.com/kubernetes-engine/docs/how-to/hardening-your-cluster)
      * ⚠️ Otherwise, the SA token can be easily exposed via Actions and do unintentional harm 🙂

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
      name: gh-runner
      annotations:
        io.kubernetes.cri-o.userns-mode: "auto:size=65536"
    spec:
      runtimeClassName: sysbox-runc
      containers:
      - name: gh-runner
        image: registry.nestybox.com/nestybox/ubuntu-bionic-systemd-docker
        command: ["/sbin/init"]
      restartPolicy: Never
    ```

4. Enter the pod and run a sample Docker rootless 🎉 container in it:
    ```
    ❯ kubectl exec gh-runner -it bash
    kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
    root@gh-runner:/# docker run --rm -it alpine
    Unable to find image 'alpine:latest' locally
    latest: Pulling from library/alpine
    29291e31a76a: Pull complete 
    Digest: sha256:eb3e4e175ba6d212ba1d6e04fc0782916c08e1c9d7b45892e9796141b1d379ae
    Status: Downloaded newer image for alpine:latest
    ```

5. If you want to [set up a custom GitHub Actions runner](https://docs.github.com/en/actions/hosting-your-own-runners/adding-self-hosted-runners):
    * Replace `${GH_REPO_URL}` `${GH_TOKEN}` with the values from your repo's _Settings/Actions/Runners/Add Runner_ page
    ```
    ❯ kubectl exec gh-runner -it bash
    root@gh-runner:/#
    apt-get update && apt-get install libdigest-sha-perl -y
    mkdir actions-runner && cd actions-runner
    curl -o actions-runner-linux-x64-2.279.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.279.0/actions-runner-linux-x64-2.279.0.tar.gz
    echo "50d21db4831afe4998332113b9facc3a31188f2d0c7ed258abf6a0b67674413a  actions-runner-linux-x64-2.279.0.tar.gz" | shasum -a 256 -c
    tar xzf ./actions-runner-linux-x64-2.279.0.tar.gz
    ./bin/installdependencies.sh
    useradd -m gh-runner -G docker
    sudo -u gh-runner ./config.sh --url ${GH_REPO_URL} --token ${GH_TOKEN}
    sudo -u gh-runner ./run.sh
    ```
    * Check this repo's [example Workflow](.github/workflows/github-workflow-demo-on-gke.yml) and the results of its run!
