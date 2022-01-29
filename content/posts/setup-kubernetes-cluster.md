---
title: "Setup Kubernetes Cluster from scratch"
date: 2022-01-28T17:09:43+11:00
draft: false
---

This post I will try to go through the steps I took to build a Kubernetes cluster from scratch.

The physcial host is an old Dell Latitude laptop with 8GB memory, which runs VMware ESXi 6.7. The plan is to run 3 nodes on it with one of the VM set as master. Each server runs Ubuntu 20.04. I will skip the VM building steps here as our focus is Kubernetes.

**Note**
Avoid installing any additional compoenents offered by Ubuntu. E.g. docker. Ubuntu uses Snap to install some of those packages, which could cause problem for us down the line.

## Linux configuration (All Nodes)
Update OS and install some required packages.
```bash
sudo apt-get update
sudo apt install ca-certificates curl gnupg lsb-release
```
## Disable Swap (All Nodes)
In order for Kubelet to work properly, we need to disable Swap on each node.
First, check if Swap is on.
```bash
sudo swapon --show
```
By default Swap is on as shown below
```
NAME       TYPE SIZE USED PRIO
/swap/file file   2G   0B   -2
```
Turn it of by run
```
sudo swapoff -a
sudo rm /swap.img
```
Disable swap on starup by comment it out in `etc/fstab`
```
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```
## Configure Iptable (All Nodes)
One prerequisite we need to set on Kubernetes nodes is to allow iptables to see bridged traffic.
First, we need to check if `br_netfilter` module is loaded.
```bash
lsmod | grep br_netfilter
```
If answer is no, load it with this command.
```bash
sudo modprobe br_netfilter
```
Next is to set `net.bridge.bridge-nf-call-iptables` to 1 in `sysctl` config.
```
sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
```
Run `sudo sysctl --system` to reload sysctl.
## Install Docker (All Nodes)
There are a few options when comes to container runtime for Kubernetes. I am personally most familiar with Docker and it tends to be easier to setup.
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo   "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
 $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io
```
Next we need to create `/etc/docker/daemon.json` with the contents below. This is to set Docker cgroup driver as systemd.

```JSON
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
```
Run these command to reload Docker
```
sudo systemctl daemon-reload
sudo systemctl restart docker
```
## Install Kubeadm and other necessities (All Nodes)
Run the commands below to install kubeadm, kubelet and kubectl.
```
sudo apt install -y apt-transport-https
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update`
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
## Initialize the control-plane node (Master Node)
Run `kubeadm init` to initialize the cluster control plane node. `192.168.0.0/16` is the network range I allocated for pod network. `10.1.1.10` is the IP address of the control plane node. It is in `10.1.1.0/24` range, which is where other worker nodes sit in.
```
 sudo kubeadm init --pod-network-cidr 192.168.0.0/16 --apiserver-advertise-address=10.1.1.10
```
Record the output with `kubeadm join` details
```
kubeadm join <control-plane-host>:<control-plane-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```
## Set permission for kubectl
This is to allow kubectl work for non-root users.
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
## Install Pod Network Addon CNI (Master Node)
If you run `kubectl get nodes` now, you will notice the master node is in `NotReady` status. This is because we have not yet provisioned the pod network.
```
$ kubectl get nodes
NAME       STATUS     ROLES                  AGE   VERSION
tom-lab1   NotReady   control-plane,master   72m   v1.23.1
```
There are a lot choices when come to pick a Pod Network Addon. In my case, I use `Weave Net` simply due to its simplicity 😅. 
To do that just run
```
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```
After a few minutes, check the node status again. This time it show as `Ready`.
```
$ kubectl get nodes
NAME       STATUS   ROLES                  AGE   VERSION
tom-lab1   Ready    control-plane,master   73m   v1.23.1
```
## Join the worker nodes (Workers Nodes)
Jump onto the Worker nodes and run the command below to join them to the cluster.
```
kubeadm join <control-plane-host>:<control-plane-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```
## Test
Run `kubectl get nodes` to check the nodes status. If you see something like below, congratulation👏🏼, you just successfully created you own Kubernetes cluster!!! Sky is the limit for you🚀!
```
NAME       STATUS   ROLES                  AGE     VERSION
tom-lab1   Ready    control-plane,master   19h     v1.23.1
tom-lab3   Ready    <none>                 7m35s   v1.23.1
```

## Troubleshooting Tips and Tricks
To reverse the kubeadm init or kubeadm join
```
sudo kubeadm reset
```
Ultimately, try a reboot of the node.

![](https://media.giphy.com/media/Oe4V14aLzv7JC/giphy.gif)

## References
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/
