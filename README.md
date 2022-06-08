# Offline-k8s-deployment





******** Online Server *********

# Install docker and cri-dockerd packages
https://download.docker.com/linux/ubuntu/dists/focal/pool/stable/amd64/

mkdir docker && cd docker/

wget https://download.docker.com/linux/ubuntu/dists/focal/pool/stable/amd64/containerd.io_1.6.6-1_amd64.deb

wget https://download.docker.com/linux/ubuntu/dists/focal/pool/stable/amd64/docker-ce-cli_20.10.9~3-0~ubuntu-focal_amd64.deb

wget https://download.docker.com/linux/ubuntu/dists/focal/pool/stable/amd64/docker-ce-rootless-extras_20.10.9~3-0~ubuntu-focal_amd64.deb

wget https://download.docker.com/linux/ubuntu/dists/focal/pool/stable/amd64/docker-ce_20.10.9~3-0~ubuntu-focal_amd64.deb

wget https://download.docker.com/linux/ubuntu/dists/focal/pool/stable/amd64/docker-compose-plugin_2.6.0~ubuntu-focal_amd64.deb

wget https://download.docker.com/linux/ubuntu/dists/focal/pool/stable/amd64/docker-scan-plugin_0.9.0~ubuntu-focal_amd64.deb

sudo apt update

sudo apt install git wget curl

visit and get a release https://github.com/Mirantis/cri-dockerd/releases

wget https://github.com/Mirantis/cri-dockerd/releases/download/v${VER}/cri-dockerd-${VER}.amd64.tgz

wget https://raw.githubusercontent.com/Mirantis/cri-dockerd/master/packaging/systemd/cri-docker.service

wget https://raw.githubusercontent.com/Mirantis/cri-dockerd/master/packaging/systemd/cri-docker.socket

cd ..

# Install CNI plugin, crictl, kubeadm, kubelet and kubectl
mkdir k8s-tools && cd k8s-tools

CNI_VERSION="v0.8.2"

ARCH="amd64"

DOWNLOAD_DIR=/usr/local/bin

CRICTL_VERSION="v1.22.0"

RELEASE="$(curl -sSL https://dl.k8s.io/release/stable.txt)"

RELEASE_VERSION="v0.4.0"

wget "https://github.com/containernetworking/plugins/releases/download/${CNI_VERSION}/cni-plugins-linux-${ARCH}-${CNI_VERSION}.tgz"

wget "https://github.com/kubernetes-sigs/cri-tools/releases/download/${CRICTL_VERSION}/crictl-${CRICTL_VERSION}-linux-${ARCH}.tar.gz"

wget https://storage.googleapis.com/kubernetes-release/release/${RELEASE}/bin/linux/${ARCH}/{kubeadm,kubelet,kubectl}

wget "https://raw.githubusercontent.com/kubernetes/release/${RELEASE_VERSION}/cmd/kubepkg/templates/latest/deb/kubelet/lib/systemd/system/kubelet.service"

wget "https://raw.githubusercontent.com/kubernetes/release/${RELEASE_VERSION}/cmd/kubepkg/templates/latest/deb/kubeadm/10-kubeadm.conf"

cd ..

# Pull Docker Images for Kubeadm

docker pull k8s.gcr.io/kube-apiserver:v1.24.1

docker pull k8s.gcr.io/kube-controller-manager:v1.24.1

docker pull k8s.gcr.io/kube-scheduler:v1.24.1

docker pull k8s.gcr.io/kube-proxy:v1.24.1

docker pull k8s.gcr.io/pause:3.7

docker pull k8s.gcr.io/pause:3.6

docker pull k8s.gcr.io/etcd:3.5.3-0

docker pull k8s.gcr.io/coredns/coredns:v1.8.6

docker pull rancher/mirrored-flannelcni-flannel

docker pull rancher/mirrored-flannelcni-flannel-cni-plugin

# Save docker images on Images Folder

mkdir images && cd images

docker save k8s.gcr.io/kube-apiserver:v1.24.1 -o apiserver.tar

docker save k8s.gcr.io/kube-controller-manager:v1.24.1 -o controller.tar

docker save k8s.gcr.io/kube-scheduler:v1.24.1 -o scheduler.tar

docker save k8s.gcr.io/kube-proxy:v1.24.1 -o proxy.tar

docker save k8s.gcr.io/pause:3.7 -o pause1.tar

docker save k8s.gcr.io/pause:3.6 -o pause2.tar

docker save k8s.gcr.io/etcd:3.5.3-0 -o etcd.tar

docker save k8s.gcr.io/coredns/coredns:v1.8.6 -o dns.tar

docker save rancher/mirrored-flannelcni-flannel -o flannel.tar

docker save rancher/mirrored-flannelcni-flannel-cni-plugin -o flannelcni.tar

cd ..

# Get Flannel conf

mkdir flannel && cd flannel

wget https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml

cd ..

# Install conntrack and socat for kubeadm

mkdir soc-conn && cd soc-conn

wget http://ftp.de.debian.org/debian/pool/main/s/socat/socat_1.7.3.1-2+deb9u1_amd64.deb

wget http://archive.ubuntu.com/ubuntu/pool/main/c/conntrack-tools/conntrack_1.4.5-2_amd64.deb

cd ..

# Copy Docker, K8s-tools,Images, flannel and soc-conn Folders from online server to offline server

scp -r docker k8s-tools images flannel soc-conn USERNAME@OFFLINE_IP_SERVER

******* Offline Server **********

# System Requirements
swapoff -a

vim /etc/fstab  =====>> Set # on swap line

cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf

overlay

br_netfilter

EOF

sudo modprobe overlay

sudo modprobe br_netfilter

// sysctl params required by setup, params persist across reboots

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf

net.bridge.bridge-nf-call-iptables  = 1

net.bridge.bridge-nf-call-ip6tables = 1

net.ipv4.ip_forward                 = 1

EOF

// Apply sysctl params without reboot

sudo sysctl --system

// Reboot system to swap conf takes the update

sudo reboot

# Install Docker and cri-dockerd

cd docker/

sudo dpkg -i *.deb

tar xvf cri-dockerd-${VER}.amd64.tgz

sudo mv cri-dockerd/cri-dockerd /usr/local/bin/

sudo mv cri-docker.socket cri-docker.service /etc/systemd/system/

sudo sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service

sudo systemctl daemon-reload

sudo systemctl enable cri-docker.service

sudo systemctl enable --now cri-docker.socket

# Load Images on system

cd images/

docker load < apiserver.tar

docker load < controller.tar

docker load < scheduler.tar

docker load < proxy.tar

docker load < pause1.tar

docker load < pause2.tar

docker load < etcd.tar

docker load < dns.tar

docker load < flannel.tar

docker load < flannelcni.tar

cd ..

# Install Kubernetes tools

cd k8s-tools/

sudo mkdir -p /opt/cni/bin

sudo tar xzf cni-plugins-linux-amd64-v0.8.2.tgz  -C /opt/cni/bin

sudo mkdir -p /usr/local/bin

sudo tar xzf crictl-v1.22.0-linux-amd64.tar.gz  -C /usr/local/bin/

sudo chmod +x {kubeadm,kubelet,kubectl}

sudo cp kubeadm kubectl kubelet /usr/bin

sudo cp kubeadm kubectl kubelet /usr/local/bin

sudo cp kubelet.service /etc/systemd/system

sudo sed -i -e 's,/usr/bin/kubelet,/usr/local/bin/kubelet,' /etc/systemd/system/kubelet.service

sudo mkdir -p /etc/systemd/system/kubelet.service.d

sudo cp 10-kubeadm.conf /etc/systemd/system/kubelet.service.d/

sudo sed -i -e 's,/usr/bin/10-kubeadm.conf,/usr/local/bin/10-kubeadm.conf,' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

sudo systemctl daemon-reload

systemctl enable --now kubelet

cd ..

# Install socat and conntrack

cd soc-conn/

sudo dpkg -i socat_1.7.3.1-2+deb9u1_amd64.deb

sudo dpkg -i conntrack_1.4.5-2_amd64.deb

cd ..

# Deploy K8s cluster

sudo kubeadm init --cri-socket /run/cri-dockerd.sock

mkdir -p $HOME/.kube

sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

sudo chown $(id -u):$(id -g) $HOME/.kube/config


//To be able to deploy pods on master node enable:

kubectl taint nodes --all node-role.kubernetes.io/control-plane- node-role.kubernetes.io/master-

# Configure Flannel

cd flannel/

kubectl patch node $(hostname) -p '{"spec":{"podCIDR":"10.100.0.1/24"}}'

kubectl apply -f kube-flannel.yml

******* Sourses *********

https://docs.docker.com/engine/install/ubuntu/#install-from-a-package

https://unix.stackexchange.com/a/181538

https://computingforgeeks.com/install-mirantis-cri-dockerd-as-docker-engine-shim-for-kubernetes/

https://packages.debian.org/stretch/amd64/socat/download

https://ubuntu.pkgs.org/20.04/ubuntu-main-amd64/conntrack_1.4.5-2_amd64.deb.html

https://github.com/flannel-io/flannel/issues/1344#issuecomment-867265435

https://download.docker.com/linux/ubuntu/dists/focal/pool/stable/amd64/

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

https://kubernetes.io/docs/setup/production-environment/container-runtimes/

https://gist.github.com/jgsqware/6595126e17afc6f187666b0296ea0723

https://github.com/flannel-io/flannel#deploying-flannel-manually

------------------------------------------------------------------------------------------------------------------------------------------------
