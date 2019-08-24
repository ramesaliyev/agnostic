[Home](../README.md)

# Kubernetes Installation

Add `yum-utils`, if not installed already;

    sudo yum install yum-utils

Add Docker repository.

    sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

List Docker repo to and choose latest version that validated by kubernetes (do a google search here).

    yum list docker-ce --showduplicates | sort -r

Install Docker CE.

    sudo yum update -y && sudo yum install -y docker-ce-18.09.8

Create `/etc/docker` directory.

    sudo mkdir /etc/docker

Setup daemon config.

    sudo vi /etc/docker/daemon.json

Paste following, save and close.

    {
      "exec-opts": ["native.cgroupdriver=systemd"],
      "log-driver": "json-file",
      "log-opts": {
        "max-size": "100m"
      },
      "storage-driver": "overlay2",
      "storage-opts": [
        "overlay2.override_kernel_check=true"
      ],
      "iptables": false
    }

Make Docker service dir.

    sudo mkdir -p /etc/systemd/system/docker.service.d

Restart Docker.

    sudo systemctl daemon-reload
    sudo systemctl enable docker.service
    sudo systemctl restart docker

Disable SELinux
> Since we are using `CentOS` we need to disable `SELinux`. This is necessary to allow containers access to the host filesystem.

    sudo setenforce 0
    sudo sed -i 's/^SELINUX=enforcing$/SELINUX=disable/' /etc/selinux/config

Disable Swap
> Swap needs to be disabled to allow kubelet to work properly.

    sudo sed -i '/swap/d' /etc/fstab
    sudo swapoff -a

Disable Firewall
> Kubernetes uses IPTables to handle inbound and outbound traffic - so to avoid any issues we disable firewalld.
> We're gonna install ufw afterwards.
> Your system may not have firewalld btw.

    sudo systemctl disable firewalld
    sudo systemctl stop firewalld

Update IPTables
> Kubernetes recommends that we ensure `net.bridge.bridge-nf-call-iptables` is set to `1`. This is due to issues where REHL/CentOS 7 has had issues with traffic being rerouted incorrectly due to bypassing iptables.

    sudo vi /etc/sysctl.d/k8s.conf

Add followings;

    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1

Reapply configs;

    sudo sysctl --system

Install `kubelet` / `kubeadm` / `kubectl`
> We will need to add the kubernetes repo to yum. Once we do that we just need to run the install command and enable kubelet.

    sudo vi /etc/yum.repos.d/kubernetes.repo

Paste following;

    [kubernetes]
    name=Kubernetes
    baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
    enabled=1
    gpgcheck=1
    repo_gpgcheck=1
    gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
    exclude=kube*

Install

    sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

Enable `kubelet`

    sudo systemctl enable --now kubelet

Now we have fully configured both our master and worker node. We can now initialize our master node and join our worker nodes to the master!