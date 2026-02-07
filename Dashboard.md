# Kubernetes Dashboard Setup Guide (Ubuntu 25+)

This guide explains how to **install and run Kubernetes Dashboard** on Ubuntu 25+, including admin access, NodePort setup, and `kubectl proxy`.

---

## 1️⃣ Prerequisites

- Kubernetes cluster installed (minikube, kind, kubeadm, or cloud)
- `kubectl` configured and working
- Internet access to pull images

Check cluster:

```bash
kubectl get nodes
kubectl version --short

```

## Deploy the Dashboard

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

# Verify pods:
kubectl get pods -n kubernetes-dashboard

# Expected output:
NAME                                        READY   STATUS    RESTARTS   AGE
dashboard-metrics-scraper-<id>             1/1     Running   0          1m
kubernetes-dashboard-<id>                  1/1     Running   0          1m

```

## Create Admin User & Token

```bash
# Create Service Account
kubectl create serviceaccount admin-user -n kubernetes-dashboard

# Bind Cluster-Admin Role
kubectl create clusterrolebinding admin-user \
    --clusterrole=cluster-admin \
    --serviceaccount=kubernetes-dashboard:admin-user

# Generate Token
kubectl -n kubernetes-dashboard create token admin-user

```

## Kubectl Proxy (Local Access)

```bash
kubectl proxy
```

wget http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

## Service not reachable:

```bash
kubectl get svc -n kubernetes-dashboard
kubectl describe svc kubernetes-dashboard -n kubernetes-dashboard
```

