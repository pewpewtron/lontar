---
id: ftuylacrafj78ugbjsfk44e
title: Installation
desc: ''
updated: 1670234311970
created: 1669705403919
---

Installing K8s on Ubuntu 20.04

prerequisite before installing

* static ip address
* hostname set for each node

## 1. Update your Ubuntu instance

```bash
sudo apt update
sudo apt -y full-upgrade
[ -f /var/run/reboot-required ] && sudo reboot -f
```

## 2. Install kubelet, kubeadm and kubectl

add Kubernetes repository for Ubuntu 20.04

```bash
sudo apt -y install curl apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Install required package

```bash
sudo apt update
sudo apt -y install vim git curl wget kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

Confirm installation by checking the version of kubectl.

```bash
kubectl version --client && kubeadm version
```

## 3. Disable swap

disable Linux swap space permanently in /etc/fstab. Search for a swap line and add `#` (hashtag) sign in front of the line.

```bash
sudo vim /etc/fstab
```

disable this line using `#`

```config
/swap.img     none    swap    sw      0       0
```

Check swap

```config
sudo swapoff -a
sudo mount -a
free -h
```

Enable kernel modules and configure sysctl.

```bash
# Enable kernel modules
sudo modprobe overlay
sudo modprobe br_netfilter

# Add some settings to sysctl
sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

# Reload sysctl
sudo sysctl --system
```

## 4. Install Container runtime

To run containers in Pods, Kubernetes uses a container runtime. Supported container runtimes are:

* Docker
* CRI-O
* Containerd

> **Note**
>
> You have to choose one runtime at a time.

### Installing Docker CE

Add repo and Install packages

```bash
sudo apt update
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install -y containerd.io docker-ce docker-ce-cli
```

Create required directories

```bash
sudo mkdir -p /etc/systemd/system/docker.service.d
```

Create daemon json config file

```bash
sudo tee /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
```

Start and enable Container runtime Services

```bash
sudo systemctl daemon-reload 
sudo systemctl restart docker
sudo systemctl enable docker
```

> **Note**
>
> For Docker Engine you need a shim interface. You can install Mirantis cri-dockerd as covered in the guide below.
>
> * [install Mirantis cri-dockerd](https://computingforgeeks.com/install-mirantis-cri-dockerd-as-docker-engine-shim-for-kubernetes/)

## 5. Initialize master node

Login to the server to be used as master and make sure that the br_netfilter module is loaded:

```bash
lsmod | grep br_netfilter

br_netfilter           28672  0
bridge                176128  1 br_netfilter
```

Enable kubelet service.

```bash
sudo systemctl enable kubelet
```

We now want to initialize the machine that will run the control plane components which includes etcd (the cluster database) and the API Server.

Pull container images:

```bash
sudo kubeadm config images pull --cri-socket unix:///run/cri-dockerd.sock
```

> **Note**
>
> These are the basic kubeadm init options that are used to bootstrap cluster.
>
> * --control-plane-endpoint :  set the shared endpoint for all control-plane nodes. Can be DNS/IP
> * --pod-network-cidr : Used to set a Pod network add-on CIDR
> * --cri-socket : Use if have more than one container runtime to set runtime socket path
> * --apiserver-advertise-address : Set advertise address for this particular control-plane node's API server

* Bootstrap with shared endpoint (DNS name for control plane API)

Set cluster endpoint DNS name or add record to /etc/hosts file.

```bash
sudo vim /etc/hosts
```

```config
172.29.20.5 k8s-master.dev
```

Create cluster:

```bash
sudo kubeadm init \
  --pod-network-cidr=192.168.0.0/16 \
  --cri-socket unix:///run/cri-dockerd.sock \
  --upload-certs \
  --control-plane-endpoint=k8s-master.dev
```

if bootstrap is complete Output will be like this;

```bash
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join k8s-master01.dev:6443 --token 86icl5.g0ufcqfos2veekiq \
        --discovery-token-ca-cert-hash sha256:8a7a20d6ac6cac2bcbe31c079ea312828495ab9e29d0b2ac788e09fab380f784 \
        --control-plane --certificate-key ae177305a43d99f1fc0b4a42dd451bf6927ddf26c5d73c7d40eaf92e8439de3d

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

  kubeadm join k8s-master01.dev:6443 \
        --token 86icl5.g0ufcqfos2veekiq \
        --discovery-token-ca-cert-hash sha256:8a7a20d6ac6cac2bcbe31c079ea312828495ab9e29d0b2ac788e09fab380f784
```

Configure kubectl using commands in the output:

```bash
mkdir -p $HOME/.kube
sudo cp -f /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Check Cluster status

```bash
kubectl cluster-info
```

## 6. Install network plugin on Master

In this guide we’ll use Calico. for other network addon you can refer here [K8s network addon](https://kubernetes.io/docs/concepts/cluster-administration/addons/)

```bash
kubectl create -f https://docs.projectcalico.org/manifests/tigera-operator.yaml 
kubectl create -f https://docs.projectcalico.org/manifests/custom-resources.yaml
```

Confirm that all of the pods are running:

```bash
kubectl get pods --all-namespaces
```

## 7. Add Worker node

add master endpoint to /etc/hosts

```config
10.8.60.167 k8s-master01.dev
```

run join command that was given is used to add a worker node to the cluster.

```bash
kubeadm join k8s-master01.dev:6443 \
  --token 86icl5.g0ufcqfos2veekiq \
  --discovery-token-ca-cert-hash sha256:8a7a20d6ac6cac2bcbe31c079ea312828495ab9e29d0b2ac788e09fab380f784
```

Run below command on the control-plane to see if the node joined the cluster.

```bash
kubectl get nodes
# or
kubectl get nodes -o wide
```

> **Note**
>
> Setting up worker node you need to follow Step 1, 2, 3, 4, above
> For adding new node to existing cluster you can follow this article
>
> * [Add new node to existing cluster](https://computingforgeeks.com/join-new-kubernetes-worker-node-to-existing-cluster/)
