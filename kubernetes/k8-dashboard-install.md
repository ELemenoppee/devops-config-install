# 👺Set Up Kubernetes Dashboard: A Comprehensive Guide

## Introduction

Looking to manage your Kubernetes cluster with an intuitive web-based interface? This detailed guide will walk you through the setup and use of the Kubernetes Dashboard, providing you with the tools to monitor and manage your cluster effectively.

By this guide, you'll be equipped to set up and use the Kubernetes Dashboard effectively, enhancing your ability to manage and monitor your Kubernetes cluster. Let's get started!

## Steps ☸️:-

**Step 1** — Login to master node

**Step 2** — Deploy the Kubernetes Dashboard using the official yaml file

```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

**Step 3** — Verify if all resources were installed successfully

```
$ kubectl get all -n kubernetes-dashboard
```

Results should be like this:

```
root@k8prod-master-01:~# kubectl get all -n kubernetes-dashboard
NAME                                             READY   STATUS    RESTARTS   AGE
pod/dashboard-metrics-scraper-5657497c4c-5bqj8   1/1     Running   0          2m13s
pod/kubernetes-dashboard-78f87ddfc-kg5xh         1/1     Running   0          2m13s

NAME                                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/dashboard-metrics-scraper   ClusterIP   10.96.127.30     <none>        8000/TCP   2m14s
service/kubernetes-dashboard        ClusterIP   10.102.157.143   <none>        443/TCP    2m14s

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/dashboard-metrics-scraper   1/1     1            1           2m13s
deployment.apps/kubernetes-dashboard        1/1     1            1           2m14s

NAME                                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/dashboard-metrics-scraper-5657497c4c   1         1         1       2m13s
replicaset.apps/kubernetes-dashboard-78f87ddfc         1         1         1       2m14s

```

**Step 4** — Modify Kubernetes Dashboard Service

```
$ kubectl edit service/kubernetes-dashboard -n kubernetes-dashboard
```

Locate the following lines and modify:

```
  type: ClusterIP -> type: NodePort
```

**Step 5** — Verify if the Kubernetes Dashboard services has been changed

```
NAME                   TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
kubernetes-dashboard   NodePort   10.102.157.143   <none>        443:30861/TCP   9m
```

**Step 6** — Create a file and a directory, then execute a kubectl command

```
$ mkdir dashboard
$ cd dashboard
$ vi k8s-serviceaccount.yaml
```

Add the following lines to k8s-serviceaccount.yaml
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dashboard-admin
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-rolebinding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: dashboard-admin
  namespace: kubernetes-dashboard
```

Then, run the kubectl command.

```
$ kubectl apply -f k8s-serviceaccount.yaml
```

**Step 7** — Generate the token

```
$ kubectl create token dashboard-admin -n kubernetes-dashboard
```

**Step 8** — Login to Kubernetes Dashboard

```
http://<MASTER_IP_ADDRESS>:<KUBERNETES_DASHBOARD_SERVICE_PORT>/
```

**Step 9** — Copy and paste the Token from Step 7 to Login.

## Final Note

If you find this repository useful for learning, please give it a star on GitHub. Thank you!

**Authored by:** [ELemenoppee](https://github.com/ELemenoppee)
