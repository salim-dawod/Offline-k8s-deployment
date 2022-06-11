# Deploy K8s Cluster using Kubeadm without internet connection

```
Deployed on Ubuntu 20:04/focal amd64 platform.

containerd.io v1.6.6-1
docker-ce-cli v20.10.9~3-0
docker-ce v20.10.9~3-0
docker-ce-rootless-extras v20.10.9~3-0
docker-compose-plugin v2.6.0
docker-scan-plugin v0.9.0

cri_dockerd v0.2.1

cni_plugin v0.8.2
crictl v1.22.0.

kube-apiserver:v1.24.1
kube-controller-manager:v1.24.1
kube-scheduler:v1.24.1
kube-proxy:v1.24.1
pause:3.7
pause:3.6
etcd:3.5.3-0
coredns:v1.8.6

socat v1.7.3.1-2
conntrack v1.4.5-2
```

**References and useful resources**

https://docs.docker.com/engine/install/ubuntu/#install-from-a-package

https://unix.stackexchange.com/a/181538

https://computingforgeeks.com/install-mirantis-cri-dockerd-as-docker-engine-shim-for-kubernetes/

https://github.com/flannel-io/flannel/issues/1344#issuecomment-867265435

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

https://kubernetes.io/docs/setup/production-environment/container-runtimes/

https://gist.github.com/jgsqware/6595126e17afc6f187666b0296ea0723

https://github.com/flannel-io/flannel#deploying-flannel-manually


**.deb packages**

https://download.docker.com/linux/ubuntu/dists/focal/pool/stable/amd64/

https://packages.debian.org/stretch/amd64/socat/download

https://ubuntu.pkgs.org/20.04/ubuntu-main-amd64/conntrack_1.4.5-2_amd64.deb.html


