---
title: "Kubernetes Cluster Fail to Start with Error: Error getting node"
date: 2023-02-11T09:27:15+11:00
draft: false

---

After a recent power outage, my Kubernetes cluster failed to come back online. I cannot connect to `kube-apiserver` through `kubectl` anymore. Upon checking the controller node, I can see these errors with `kubelet` service.

```log
Feb 03 00:43:06 tom-lab1 kubelet[3475]: E0203 00:43:06.555186    3475 kubelet.go:2422] "Error getting node" err="node \"tom-lab1\" not found"
Feb 03 00:43:06 tom-lab1 kubelet[3475]: E0203 00:43:06.656323    3475 kubelet.go:2422] "Error getting node" err="node \"tom-lab1\" not found"
```

In `kube-apiserver` logs `/var/log/container/kube-apiserver-tom-lab1_kube-system_kube-apiserver-2bec70209c1231c69a6501aea951f4ff5bed1996174028cd7e0396b2c4dc34e0.log` I can see the errors below.
```log
{"log":"W0203 02:07:53.990782       1 clientconn.go:1331] [core] grpc: addrConn.createTransport failed to connect to {127.0.0.1:2379 127.0.0.1 \u003cnil\u003e 0 \u003cnil\u003e}. Err: connection error: desc = \"transport: authentication handshake failed: x509: certificate has expired or is not yet valid: current time 2023-02-03T02:07:53Z is after 2023-01-27T03:03:26Z\". Reconnecting...\n","stream":"stderr","time":"2023-02-03T02:07:53.990925842Z"}
```

So clearly the certificate for my kube-apiserver somehow expired without auto renew itself.

Here are the steps to fix this problem by renewing the certificates.
## On the controller node

### Backup existing certificates
```bash
cp /etc/kubenetes/pki/*.key /etc/kubenetes/pki/*.key.old
cp /etc/kubenetes/pki/*.crt /etc/kubenetes/pki/*.crt.old
```
### Renew all certificates
```bash
kubeadm certs check-expiration
kubeadm certs renew all
```
### Restrat kubelet service
```bash
systemctl kubelet restart
```
### Set Kubernetes to use the new certificates
```bash
kubeadm  init phase kubeconfig all
```
### Restart the master components
```bash
sudo kill -s SIGHUP $(pidof kube-apiserver)
sudo kill -s SIGHUP $(pidof kube-controller-manager)
sudo kill -s SIGHUP $(pidof kube-scheduler)
```
### Check if the certificate is renewed
```bash
echo -n | openssl s_client -connect localhost:10257 2>&1 | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' | openssl x509 -text -noout | grep Not
```

> ⚠️**NOTE**
> 
> My cluster version is 1.23.3.
> 
> Kubeadm version is 1.23.1.
> 
> In certain scenario, you might have to re-join the worker nodes to the cluster again. 


## References
https://github.com/kubernetes/kubeadm/issues/581#issuecomment-421477139
