---
title: "Run Pihole on Kubernetes"
date: 2022-10-12T16:33:35+11:00
draft: false
---

In this post I will show how I deployed Pihole onto my local Kubernetes cluster. In compare with some other online tutorials, this deployment saves Pihole configuration onto persistent volumes, so you won't need to reconfigure everything after pod recreation. By using LoadBalancer, original client IPs are captured correctly in my Pihole instead of seeing everything was from the Cluster IP.

> If you want to learn how to build your own local Kubernetes cluster, read [this post](https://www.tomking.xyz/posts/setup-kubernetes-cluster/).

The overall architecture looks like this.
![](https://blogfilesr2.tomking.xyz/pihole-on-k8s-architect.png)

First we need to deploy a load balancer to expose our Pihole ports (http and dns) to the local network outside the cluster. For that purpose we use MetalLB. 
> If your cluster is hosted in Cloud like GCP, AWS or Azure, you do not need/should not use MetalLB, as these hosted Kubernetes cluster will come with load balancers pre-configured.

MelLB deployment is very simple. All you need is the manifest for MetalLB plus the configuration value file.

```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.5/config/manifests/metallb-native.yaml
```

>Make sure your cluster version is compatible with the version of Metallb you are installing.
>To check your cluster version run `kubectl version` and check **Server Version**.

Next, deploy the config value file below. The file set address range for the load balancer.
```
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb-system
spec:
  addresses:
  - 10.1.1.18-10.1.1.20 # IP range reserved for the Load balancer
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: tomlab
  namespace: metallb-system
---
```

After successfully deploy the load balancer, you are now ready to deploy the pod for Pihole.

The manifest for Pihole deployment can be found [here](https://raw.githubusercontent.com/tomkingchen/pihole-kubernetes/main/pihole.yaml).

The manifest creates bunch of resources. The ones worth noting are the 2 persistent volumes. They are created `local` to node `tom-lab3`.
```yaml
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - tom-lab3
```

To publish the Pihole pod, we use two separate services via different IPs. One for UDP ports and one for TCP.
- Service: pihole-udp (10.1.1.20)
- Service: pihole (10.1.1.19)

> In your environment, please use the spare IPs assigned to the `MetalLB` load balancer. For example in my lab, 10.1.1.18 is already assigned to Istio gateway. So I cannot use that address for Pihole services.

If everything deployed successfully, you should be able to visit http://10.1.1.19/admin and see the pretty Pihole login page. 

> Remember to change the current password, as it is for everyone to see in the manifest file.

![](https://blogfilesr2.tomking.xyz/pihole-screenshot.png)
