---
title: "Setup Kubernetes Cluster TL;DR version"
date: 2022-01-28T17:09:43+11:00
draft: true
---

This post details the steps I took to build a Kubernetes cluster from scratch.

The physcial host is an old Dell Latitude laptop with 8GB memory, which runs VMware ESXi 6.7. The plan is to run 3 nodes on it with one of the VM set as master. Each server runs Ubuntu 20.04. I will skip the VM building steps here as our focus is Kubernetes.

**On all three nodes**
## Linux Configuration 
Update OS with the usual 
```bash
sudo apt-get update
```
Install basic tools like 

Disable Swap
Iptable

## Install Docker
Install runtime
Create /etc/docker/daemon.json

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
Restart Docker daemon and kubelet


## Install Kubeadm



## Configure Network Addon CNI
Weave Net


## Bootstrap the master node


## Join the worker nodes

