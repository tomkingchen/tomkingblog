---
title: "Kubernetes Deploy Metrics Server With Tls"
date: 2022-11-10T13:50:40+11:00
draft: false
---

Metrics-server is one of the most common service deployed in their Kubernetes clusters. It is designed to be used for autoscaling purposes. In my case, I simply want to have a way to easily check my nodes status with `kubectl top node`.

In this post I will walk you through the steps I took to get `metrics-server` deployed and working on a local Kubernetes cluster. There are a few interesting issues I bumped into along the way. I will show you how I did troubleshooting and solved them.

Some information about the environment I used for this deployment.
- Kubernetes cluster version 1.23.3
- 3 nodes cluster (1 controller + 2 workers)
- Metrics server version is v0.6.1
- Use Weavenet as CNI

First of all, run the command below to deploy metrics-server with the latest manifest.

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

## Enabling Signed Kubelet Serving Certificates`
By default metrics-server requires kubelet certificates to be signed by the *cluster CA*. As all my nodes were using self-signed certificate, I got the errors below when checking logs on the metrics-server pod.

```bash
➜ kubectl logs metrics-server-xxxx -n kube-system
...
E1107 05:54:46.914203       1 scraper.go:140] "Failed to scrape node" err="Get \"https://10.1.1.162:10250/metrics/resource\": x509: cannot validate certificate for 10.1.1.162 because it doesn't contain any IP SANs" node="tom-lab2"
```
Well you can simply disable TLS for metrics-server to avoid the error, but since that should never be done to a production cluster, let's not do that and enable signed kubelet serving certificates instead.
1. Check kubelet `ConfigMap` by running the command below (Your version might be different to mine, so just change the version number accordingly in your command). Confirm `serverTLSBootstrap: true` is not present. If it's already there, go to step 3.
```bash
➜ kubectl get cm kubelet-config-1.23 -n kube-system
```
2. Run the command below to edit the kubelet-config-1.23 (should be your own cluster version here) ConfigMap in the kube-system namespace. In the ConfigMap, add `serverTLSBootstrap: true`. `kubectl edit` works just like normal [vim](https://en.wikipedia.org/wiki/Vim_(text_editor)). Insert `serverTLSBootstrap: true` just underneath line `kind: KubeletConfiguration`. Save the file with `SHIFT+zz`.
```bash
➜ kubectl edit configmap kubelet-config-1.23 -n kube-system
```
3. Check ConfigMap after the change and make sure `serverTLSBootstrap` is added.
```bash
➜ kubectl get cm kubelet-config-1.23 -n kube-system -o yaml
```
Output should look like this, make sure the indentation is correct:
```yaml
apiVersion: v1
data:
  kubelet: |
    apiVersion: kubelet.config.k8s.io/v1beta1
    authentication:
      anonymous:
        enabled: false
...
    imageMinimumGCAge: 0s
    kind: KubeletConfiguration
    serverTLSBootstrap: true
    logging:
      flushFrequency: 0
...
    memorySwap: {}
...
    volumeStatsAggPeriod: 0s
kind: ConfigMap
metadata:
  annotations:
    kubeadm.kubernetes.io/component-config.hash: sha256:000a5dd81630f64bd6f310899359dfb9e8818fc8fe011b7fdc0ec73783a42452
  creationTimestamp: "2022-01-27T03:03:39Z"
...
  uid: f4303bd7-f1e7-41e6-8954-172283c279ab
```
4. Next, SSH to each of the nodes and edit kubelet config file `/var/lib/kubelet/config.yaml`. Add `serverTLSBootstrap: true` to the config. It should be in the same position as in the ConfigMap.
5. After change the config file, restart kubelet by running `systemctl restart kubelet`. You will obviously need ROOT priviledge to restart it.
6. Once the kubelet on the nodes are restarted, you should see bunch CSRs are being created. If not, please double check the config file and make sure there is no typo or indentation errors with the file.
```bash
➜ kubectl get csr
NAME        AGE   SIGNERNAME                                    REQUESTOR              REQUESTEDDURATION   CONDITION
csr-86v4c   50s   kubernetes.io/kube-apiserver-client-kubelet   system:node:tom-lab2   <none>              Approved,Issued
csr-zfn8q   49s   kubernetes.io/kubelet-serving                 system:node:tom-lab2   <none>              Pending
```
7. Approve them with this command
```bash
➜ kubectl certificate approve <CSR-name>
```
8. Check CSRs status again and you should see they are now approved.
```bash
➜ kubectl get csr
NAME        AGE     SIGNERNAME                                    REQUESTOR              REQUESTEDDURATION   CONDITION
csr-86v4c   2m43s   kubernetes.io/kube-apiserver-client-kubelet   system:node:tom-lab2   <none>              Approved,Issued
csr-zfn8q   2m42s   kubernetes.io/kubelet-serving                 system:node:tom-lab2   <none>              Approved,Issued
```
Now kubelet on your nodes are running with certificates signed by the cluster CA.

## Enable Aggregator Routing
`kubectl top node` still didn't work after the certificate change. I got `the server is currently unable to handle the request` error from running the command. Checking `kube-apiserver` pod logs, I got error below.
```bash
➜ kubectl logs kube-apiserver-tom-lab1 -nkube-system
...
E1109 09:25:11.288855       1 available_controller.go:524] v1beta1.metrics.k8s.io failed with: failing or missing response from https://10.97.189.231:443/apis/metrics.k8s.io/v1beta1: Get "https://10.97.189.231:443/apis/metrics.k8s.io/v1beta1": dial tcp 10.97.189.231:443: connect: no route to host
```
Based on [this issue](https://github.com/kubernetes-sigs/metrics-server/issues/448), the error indicates kube-apiserver cannot route to the metrics api, possible cause could be that *aggreation-layer-routing* is disabled.

Here is [the documentation](https://kubernetes.io/docs/tasks/extend-kubernetes/configure-aggregation-layer/) talks about how to configure this feature. Below steps show how I get it configured. 

First, SSH to the control node, and check current `kube-apiserver` startup parameters.
```bash
➜ cat /etc/kubernetes/manifests/kube-apiserver.yaml
```
Here is the output. As you can see I have all necessary command flags described in the document. So there is no changes need to be made in my case.
```yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 10.1.1.41:6443
  creationTimestamp: null
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=10.1.1.41
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://127.0.0.1:2379
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
    - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
    - --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
    - --requestheader-allowed-names=front-proxy-client
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --requestheader-extra-headers-prefix=X-Remote-Extra-
    - --requestheader-group-headers=X-Remote-Group
    - --requestheader-username-headers=X-Remote-User
    - --secure-port=6443
    - --service-account-issuer=https://kubernetes.default.svc.cluster.local
    - --service-account-key-file=/etc/kubernetes/pki/sa.pub
    - --service-account-signing-key-file=/etc/kubernetes/pki/sa.key
    - --service-cluster-ip-range=10.96.0.0/12
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    image: k8s.gcr.io/kube-apiserver:v1.23.3
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 10.1.1.41
        path: /livez
        port: 6443
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: kube-apiserver
    readinessProbe:
      failureThreshold: 3
      httpGet:
        host: 10.1.1.41
        path: /readyz
        port: 6443
        scheme: HTTPS
      periodSeconds: 1
      timeoutSeconds: 15
    resources:
      requests:
        cpu: 250m
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 10.1.1.41
        path: /livez
        port: 6443
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /etc/ssl/certs
      name: ca-certs
      readOnly: true
    - mountPath: /etc/ca-certificates
      name: etc-ca-certificates
      readOnly: true
    - mountPath: /etc/pki
      name: etc-pki
      readOnly: true
    - mountPath: /etc/kubernetes/pki
      name: k8s-certs
      readOnly: true
    - mountPath: /usr/local/share/ca-certificates
      name: usr-local-share-ca-certificates
      readOnly: true
    - mountPath: /usr/share/ca-certificates
      name: usr-share-ca-certificates
      readOnly: true
  hostNetwork: true
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  volumes:
  - hostPath:
      path: /etc/ssl/certs
      type: DirectoryOrCreate
    name: ca-certs
  - hostPath:
      path: /etc/ca-certificates
      type: DirectoryOrCreate
    name: etc-ca-certificates
  - hostPath:
      path: /etc/pki
      type: DirectoryOrCreate
    name: etc-pki
  - hostPath:
      path: /etc/kubernetes/pki
      type: DirectoryOrCreate
    name: k8s-certs
  - hostPath:
      path: /usr/local/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-local-share-ca-certificates
  - hostPath:
      path: /usr/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-share-ca-certificates
status: {}
```
My `metrics-server-xxxx` pod is running on worker node instead of the control node itself, which is not the host `kube-apiserver` pod is running on. So I will need to set flag `--enable-aggregator-routing = true` in the kube-apiserver manifest. So just add `--enable-aggregator-routing=true` as one of the command flags in /etc/kubernetes/manifests/kube-apiserver.yaml file.

## Fix Network Configuration
Frustratingly, metrics-server still refuse to work after enabling aggregator-routing.
Logs from `metrics-server` pod
```bash
E1109 09:44:38.039028       1 scraper.go:140] "Failed to scrape node" err="request failed, status: \"500 Internal Server Error\"" node="tom-lab2"
```
Logs from `kube-apiserver` pod
```bash
E1110 00:18:23.386884       1 available_controller.go:524] v1beta1.metrics.k8s.io failed with: failing or missing response from https://10.47.0.12:4443/apis/metrics.k8s.io/v1beta1: Get "https://10.47.0.12:4443/apis/metrics.k8s.io/v1beta1": dial tcp 10.47.0.12:4443: connect: no route to host
```
Based on the error, it appears kube-apiserver tries to reach metrics-server via its pod IP address. The kube-apiserver uses NodeIP which is in a completely different subnet. There is no route between the two network. This obviously won't work. In order to fix the issue, I need to configure metrics-server to use NodeIP as well. To do that, I simply add these two lines to metrics manifest file (The file is downloaded from https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml)
```yaml
    hostNetwork: true
    dnsPolicy: ClusterFirstWithHostNet
```
Apply the manifest `kubectl apply -f ./components.yaml`. and that's it 🎉!!! I can finally see node resource usage with `kubectl top node` command.

```bash
➜ kubectl top node
NAME       CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
tom-lab1   204m         10%    1345Mi          71%
tom-lab2   254m         12%    1202Mi          63%
tom-lab3   299m         14%    1334Mi          70%
```