# Kubernetes Cluster Creation with Kubeadm and Vagrant

![Kubernetes Cluster Creation with Kubeadm and Vagrant - Thumbnail](/architecture-diagram/YT-Thumbnail%20- Deploy%20Kubernetes%20Cluster%20with%20Kubeadm.png)

---

## Architecture Diagram

![Kubernetes Cluster Creation with Kubeadm and Vagrant - Architecture](/architecture-diagram/Kubernetes-Architecture.svg)

---

## Introduction

Welcome to the comprehensive video tutorial on setting up a Kubernetes Cluster Creation with Kubeadm and Vagrant! Are you ready to dive into the world of container orchestration and streamline your application deployment process? Look no further, as this tutorial will guide you through the entire journey, from provisioning virtual machines to running your first Kubernetes application.

## Problem Statement

Modern application development demands scalability, flexibility, and efficiency. Kubernetes has emerged as the de facto solution for container orchestration, but setting up a Kubernetes cluster from scratch can be a daunting task, especially for beginners. Traditional cloud-based solutions like Amazon EKS might seem like the easy way out, but they can be costly, and you might prefer the control and cost savings that come with managing your own infrastructure.

This is where our tutorial comes to the rescue. We understand the need for an accessible, cost-effective, and educational solution to set up a Kubernetes cluster using Kubeadm, a reliable and versatile tool. This tutorial addresses the challenges faced by developers and system administrators who want to harness the power of Kubernetes on their own terms.

## Solution

Our video tutorial offers a step-by-step guide to creating a Kubernetes cluster using Kubeadm and Vagrant. We break down the process into manageable chunks, ensuring that you can follow along even if you are new to Kubernetes. Here's what you can expect from this tutorial:

1. **Provisioning Virtual Machines:** Learn how to create two virtual machines for your Kubernetes cluster using Vagrant, making it easy to set up a test environment on your local machine.

2. **Downloading and Installing Kubernetes Binaries:** We guide you through the process of obtaining and installing the essential Kubernetes binaries to get your cluster up and running.

3. **Container Runtime Components:** Understand the container runtime components required for Kubernetes and learn how to download and install them.

4. **Configuration and Starting the Container Runtime:** Configure the container runtime to work seamlessly with Kubernetes and ensure it's up and running.

5. **Networking Features:** Enable essential networking features within your cluster to ensure smooth communication between pods and nodes.

6. **Starting the Control Plane and Installing Antrea:** Get the control plane up and running on the control plane node and install Antrea for networking and security enhancements.

7. **Joining Worker Nodes:** Learn how to add worker nodes to your cluster to distribute the workload effectively.

8. **Testing the Cluster:** Finally, test your Kubernetes cluster by deploying an example application, solidifying your understanding of the setup.

9. **Kubeadm Clean Up:** Clean up your cluster when you're done experimenting, ensuring a fresh start for future endeavors.

By the end of this tutorial, you will have a fully functional Kubernetes cluster that you can use for development, testing, and learning purposes. Gain the skills and confidence to manage your Kubernetes clusters like a pro, all while keeping full control of your infrastructure.

Join us on this journey towards Kubernetes mastery with Kubeadm and Vagrant, and empower yourself to build and deploy applications at scale with ease.

**Get started now!** 🚀

Please note that this tutorial focuses on using Kubeadm as the hero tool for setting up the Kubernetes cluster, giving you a deeper understanding of the inner workings of Kubernetes. It's an empowering way to take your container orchestration skills to the next level.

---

## Create Virtual Machines for Kubernetes Cluster

```shell
vagrant --version
vagrant up

vagrant status

vagrant upload ./vm.node-a.network node-a
vagrant upload ./vm.node-b.network node-b
```

### node-a (different terminal)

```shell
vagrant ssh node-a

sudo hostnamectl set-hostname node-a

sudo mv ./vm.node-a.network /etc/systemd/network
sudo systemctl restart systemd-networkd

# ping node-b
ping 192.168.56.102
```

### node-b (different terminal)

```shell
vagrant ssh node-b

sudo hostnamectl set-hostname node-b

sudo mv ./vm.node-b.network /etc/systemd/network
sudo systemctl restart systemd-networkd

# ping node-a
ping 192.168.56.101
```

---

## Download and Install Kubernetes binaries

### node-a (different terminal)

```shell
version=$(curl -L https://dl.k8s.io/release/stable.txt)
echo $version

# Install kubeadm, kubelet, kubectl
sudo curl -L -o /usr/local/bin/kubeadm https://dl.k8s.io/release/$version/bin/linux/amd64/kubeadm
sudo curl -L -o /usr/local/bin/kubelet https://dl.k8s.io/release/$version/bin/linux/amd64/kubelet
sudo curl -L -o /usr/local/bin/kubectl https://dl.k8s.io/release/$version/bin/linux/amd64/kubectl

# give executable permission
sudo chmod +x /usr/local/bin/kube*

# check versions
kubeadm version
kubelet --version
kubectl version

# exit from ssh terminal
exit

# upload files to node-a
vagrant upload kubeadm.node-a.conf node-a
vagrant upload k8s.conf node-a
vagrant upload kubelet.service node-a

# ssh into node-a
vagrant ssh node-a

sudo mkdir -p /etc/systemd/system/kubelet.service.d
sudo cp kubeadm.node-a.conf /etc/systemd/system/kubelet.service.d

sudo cp kubelet.service /etc/systemd/system

sudo systemctl daemon-reload
sudo systemctl enable --now kubelet
sudo systemctl status kubelet
```

### node-b (different terminal)

```shell
version=$(curl -L https://dl.k8s.io/release/stable.txt)
echo $version

# Install kubeadm, kubelet, kubectl
sudo curl -L -o /usr/local/bin/kubeadm https://dl.k8s.io/release/$version/bin/linux/amd64/kubeadm
sudo curl -L -o /usr/local/bin/kubelet https://dl.k8s.io/release/$version/bin/linux/amd64/kubelet
sudo curl -L -o /usr/local/bin/kubectl https://dl.k8s.io/release/$version/bin/linux/amd64/kubectl

# give executable permission
sudo chmod +x /usr/local/bin/kube*

# check versions
kubeadm version
kubelet --version
kubectl version

# exit from ssh terminal
exit

# upload files to node-b
vagrant upload kubeadm.node-b.conf node-b
vagrant upload k8s.conf node-b
vagrant upload kubelet.service node-b 

# ssh into node-b
vagrant ssh node-b

sudo mkdir -p /etc/systemd/system/kubelet.service.d
sudo cp kubeadm.node-b.conf /etc/systemd/system/kubelet.service.d

sudo cp kubelet.service /etc/systemd/system

sudo systemctl daemon-reload
sudo systemctl enable --now kubelet
sudo systemctl status kubelet
```

---

## Download the Container Runtime Components

### node-a (different terminal)

```shell
# visit `https://github.com/containerd/containerd/` > releases page > copy link of `linux-amd64.tar.gz` version
curl -L https://github.com/containerd/containerd/releases/download/v1.7.5/containerd-1.7.5-linux-amd64.tar.gz -o /tmp/containerd.tar.gz

# verify that the `containerd` tarball is there.
ls /tmp

# visit `https://github.com/containerd/containerd/` and look for `containerd.service` file > click `Raw` > copy the url 
sudo curl -L -o /etc/systemd/system/containerd.service https://raw.githubusercontent.com/containerd/containerd/main/containerd.service

# visit `https://github.com/kubernetes-sigs/cri-tools` > releases page > copy link of `*-linux-amd64.tar.gz` version
curl -L https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.28.0/crictl-v1.28.0-linux-amd64.tar.gz -o /tmp/crictl.tar.gz

# visit `https://github.com/containernetworking/plugins/` > releases page > copy link of `*-linux-amd64.tgz` version
curl -L https://github.com/containernetworking/plugins/releases/download/v1.3.0/cni-plugins-linux-amd64-v1.3.0.tgz -o /tmp/cni.tar.gz
```

### node-b (different terminal)

```shell
# do to containerd/containerd releases page, and copy link of `linux-amd64` version
curl -L https://github.com/containerd/containerd/releases/download/v1.7.5/containerd-1.7.5-linux-amd64.tar.gz -o /tmp/containerd.tar.gz

# verify that the `containerd` tarball is there.
ls /tmp

# visit `https://github.com/containerd/containerd/` and look for `containerd.service` file > click `Raw` > copy the url 
sudo curl -L -o /etc/systemd/system/containerd.service https://raw.githubusercontent.com/containerd/containerd/main/containerd.service

# visit `https://github.com/kubernetes-sigs/cri-tools` > releases page > copy link of `linux-amd64.tar.gz` version
curl -L https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.28.0/crictl-v1.28.0-linux-amd64.tar.gz -o /tmp/crictl.tar.gz

# visit `https://github.com/containernetworking/plugins/` > releases page > copy link of `*-linux-amd64.tgz` version
curl -L https://github.com/containernetworking/plugins/releases/download/v1.3.0/cni-plugins-linux-amd64-v1.3.0.tgz -o /tmp/cni.tar.gz
```

---

## Install the Container Runtime Components

### node-a (different terminal)

```shell
# Installing containerd
pushd /usr/local
sudo tar -xvf /tmp/containerd.tar.gz
popd

containerd --version 

# Installing crictl
pushd /usr/local/bin
sudo tar -xvf /tmp/crictl.tar.gz
popd

crictl --version

# Installing cni-plugins
sudo mkdir -p /opt/cni/bin
pushd /opt/cni/bin
sudo tar -xvf /tmp/cni.tar.gz
popd

```

### node-b (different terminal)

```shell
# Installing containerd
pushd /usr/local
sudo tar -xvf /tmp/containerd.tar.gz
popd

containerd --version

# Installing crictl
pushd /usr/local/bin
sudo tar -xvf /tmp/crictl.tar.gz
popd

crictl --version

# Installing cni-plugins
sudo mkdir -p /opt/cni/bin
pushd /opt/cni/bin
sudo tar -xvf /tmp/cni.tar.gz
popd
```

---

## Configure the Container Runtime

### node-a (different terminal)

```shell
# download runc
sudo apt -y update
sudo apt -y install runc

sudo mkdir -p /etc/containerd
containerd config default > /tmp/config.toml
cat /tmp/config.toml
sudo mv /tmp/config.toml /etc/containerd

# make change in /etc/containerd/config.toml > `SystemdCgroup = True`
grep -i systemd /etc/containerd/config.toml
# shows what it will change
sudo sed 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml | grep Systemd
# verify that it has not modified the file yet
grep -i systemd /etc/containerd/config.toml
# `-i` make changes to the file
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
# check again that the changes will made in the file
grep -i systemd /etc/containerd/config.toml
```

### node-b (different terminal)

```shell
# download runc
sudo apt -y update
sudo apt -y install runc

sudo mkdir -p /etc/containerd
containerd config default > /tmp/config.toml
cat /tmp/config.toml
sudo mv /tmp/config.toml /etc/containerd

# make change in /etc/containerd/config.toml > `SystemdCgroup = True`
grep -i systemd /etc/containerd/config.toml
# shows what it will change
sudo sed 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml | grep Systemd
# verify that it has not modified the file yet
grep -i systemd /etc/containerd/config.toml
# `-i` make changes to the file
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
# check again that the changes will made in the file
grep -i systemd /etc/containerd/config.toml
```

---

## Start the Container Runtime

### node-a (different terminal)

```shell
# reload systemctl
sudo systemctl daemon-reload
sudo systemctl enable --now containerd
sudo systemctl status containerd

# test that `containerd` is working
sudo ctr image pull docker.io/library/hello-world:latest
# start the container
sudo ctr run --rm docker.io/library/hello-world:latest test
```

### node-b (different terminal)

```shell
# reload systemctl
sudo systemctl daemon-reload
sudo systemctl enable --now containerd
sudo systemctl status containerd

# test that `containerd` is working
sudo ctr image pull docker.io/library/hello-world:latest
# start the container
sudo ctr run --rm docker.io/library/hello-world:latest test
```

---

## Enable Networking Features

### node-a (different terminal)

```shell
hostname

# reset kubeadm
sudo kubeadm reset -f

#install packages
sudo apt -y install socat conntrack

# turn off swap
sudo swapoff -a && sudo systemctl mask swap.img.swap

# k8s conf
sudo touch /etc/modules-load.d/k8s.conf /etc/sysctl.d/k8s.conf

# edit k8s
sudo vim /etc/modules-load.d/k8s.conf
```

```conf
overlay
br_netfilter
```

```shell
sudo vim /etc/sysctl.d/k8s.conf
```

```conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
```

```shell
# reboot terminal
sudo reboot
```

### node-b (different terminal)

```shell
#install packages
sudo apt -y install socat conntrack

# turn off swap
sudo swapoff -a && sudo systemctl mask swap.img.swap

# k8s conf
sudo touch /etc/modules-load.d/k8s.conf /etc/sysctl.d/k8s.conf

# edit k8s
sudo vim /etc/modules-load.d/k8s.conf
```

```conf
overlay
br_netfilter
```

```shell
sudo vim /etc/sysctl.d/k8s.conf
```

```conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
```

```shell
# reboot terminal
sudo reboot
```

---

## Start the Control Plane on the Control Plane Node

### node-a (different terminal)

```shell
vagrant ssh node-a

sudo kubeadm init phase preflight

sudo kubeadm init --apiserver-advertise-address 192.168.56.101 --pod-network-cidr 100.64.0.0/16

sudo kubectl --kubeconfig /etc/kubernetes/admin.conf get nodes

# visit `https://github.com/antrea-io/antrea` > click on `releases` > under `Assets` > copy link address `https://github.com/antrea-io/antrea/releases/download/v1.13.0/antrea.yml`
sudo kubectl --kubeconfig /etc/kubernetes/admin.conf apply -f  https://github.com/antrea-io/antrea/releases/download/v1.13.0/antrea.yml

sudo kubectl --kubeconfig /etc/kubernetes/admin.conf get nodes

```

### node-b (different terminal)

```shell
vagrant ssh node-b
```

---

## Join the Worker Node to the Control Plane Node

### node-a (different terminal)

> **_NOTE:_**  From above in the terminal and copy the `kubeadm join ...` command.

```shell
# execute this command after the `kubeadm join` command is executed in the `node-b` terminal.
sudo kubectl --kubeconfig /etc/kubernetes/admin.conf get nodes
```

### node-b (different terminal)

> **_NOTE:_**  Paste `kubeadm join ...` command in the `node-b` terminal.

```shell
sudo kubeadm join ...
```

---

## Test the Cluster with and Example App

### node-a (different terminal)

```shell
sudo kubectl --kubeconfig /etc/kubernetes/admin.conf run --rm --stdin --image=hello-world --restart=Never --request-timeout=30 test-pod

```

---

## Kubeadm Clean up

```shell
# exit out of both the terminal sessions
exit

# destroy vagrant VMs
vagrant destroy
```

---
