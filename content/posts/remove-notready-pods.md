---
title: "How to remove NotReady Pods from Kubernetes"
date: 2022-10-12T16:36:24+11:00
draft: false
---

## Problem
You got a pod left in `NotReady` status due to failed helm chart installation or resource deployment.
```
➜  ✗ kubectl get pods
NAME                                                READY   STATUS     RESTARTS   AGE
release-0.11-kube-promethe-admission-create-7j7fz   1/2     NotReady   2          8m1s
```
Tried to remove the pod with `kubectl delete pod` will just cause the pod to be recreated.


## Solutions
### Uninstall helm chart
Run `helm ls -A` to list all current installed charts. Check the chart status.
```
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS  CHART                           APP VERSION
tomlab          default         1               2022-09-26 09:30:38.909619 +1000 AEST   failed  kube-prometheus-stack-40.1.2    0.59.1 
```

To remove the failed chart, run `helm uninstall [chart_name] -n [namespace_name]`. This should remove the pods created by the chart.

### Check Pending Deployments
Sometimes a pending deployment will left pods in NotReady status. As long as the deployment is not removed, the pods will get recreated even when you try to delete them manually.

To check currently deployments:
```
kubectl get deployments --all-namespaces
```

To remove the deployments and its resources:
```
kubectl delete deployment [deployment_name]
```

### Check Pending Jobs
If all above fails, lookup all the jobs currently pending.
```
➜ ✗ kubectl get job
NAME                                          COMPLETIONS   DURATION   AGE
release-0.11-kube-promethe-admission-create   0/1           2d17h      2d17h
```

Kill the job
```
kubectl delete -f jobs [job_name]
```
After that you should be able to clean up the `NotReady` pods relate to the job.