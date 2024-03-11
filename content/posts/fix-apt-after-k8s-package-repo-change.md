---
title: "Fix Ubuntu APT Update Error After Kubernetes Package Repo Change"
date: 2024-03-11T11:25:27+11:00
draft: false
---

Kubernetes legacy package repositories (apt.kubernetes.io and yum.kubernetes.io) have been deprecated and frozen starting from September 13, 2023.  It is strongly recommended to use the new package repositories hosted at pkgs.k8s.io. It is also required in order to install Kubernetes versions released after September 13, 2023. The new package repositories provide downloads for Kubernetes versions starting with v1.24.0. Ubuntu packages update will fail with `404 Not Found` error if you have Kubernetes packages still point to the deprecated repositories.

```
$ sudo apt update
Hit:1 http://au.archive.ubuntu.com/ubuntu jammy InRelease
Hit:2 http://au.archive.ubuntu.com/ubuntu jammy-updates InRelease
Hit:3 http://au.archive.ubuntu.com/ubuntu jammy-backports InRelease
Hit:4 http://au.archive.ubuntu.com/ubuntu jammy-security InRelease
Hit:5 https://download.docker.com/linux/ubuntu jammy InRelease
Ign:6 https://packages.cloud.google.com/apt kubernetes-xenial InRelease
Err:7 https://packages.cloud.google.com/apt kubernetes-xenial Release
  404  Not Found [IP: 142.250.70.238 443]
Reading package lists... Done
E: The repository 'https://apt.kubernetes.io kubernetes-xenial Release' no longer has a Release file.
N: Updating from such a repository can't be done securely, and is therefore disabled by default.
N: See apt-secure(8) manpage for repository creation and user configuration details.
```

To fix the issue, simply remove the old repo source list and replace it with the new `k8s.io` repo.

First, remove the old `kuberntes.list`
```Bash
sudo rm /etc/apt/sources.list.d/kuberntes.list
```

Second, create new `kubernetes.list` file. One thing to note here is you have to specify the **CURRENT** version of Kubernetes on the node. In my case, I got `1.26` running on the node. So here's the command to create the file.
```Bash
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \https://pkgs.k8s.io/core:/stable:/v1.26/deb/ /" \| sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Last, you need to download the public signing key for the Kubernetes package repositories.
```Bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.26./deb/Rlease.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-ap-keyring.gpg
```

That's it. Now you should be able to run `sudo apt update` without errors.
