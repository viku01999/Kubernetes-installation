# ☸️ Kubernetes Hands-On Guide (Beginner → Confident)

This repository is a **first-time, practical Kubernetes guide** for learning by doing.
You will deploy applications, access them, understand networking, and clean up resources properly.

---

## 📌 Environment

- OS: Ubuntu (Local system)
- Kubernetes installed using `kubeadm`
- Container Runtime: containerd
- CNI: Calico
- Access method: `kubectl` + Kubernetes Dashboard (via kube-proxy)

---

## 🧠 Kubernetes Core Concepts (Simple Words)

| Term | Meaning |
|----|----|
| Cluster | A group of machines running Kubernetes |
| Node | A machine (VM or physical) in the cluster |
| Pod | Smallest unit that runs containers |
| Deployment | Manages Pods (restart, scale, update) |
| Service | Network access to Pods |
| Namespace | Logical separation inside cluster |

---

## 👉 Restart kubernetes
```bash
sudo systemctl daemon-reexec
sudo systemctl restart containerd
sudo systemctl restart kubelet

# Check kubelet status
sudo systemctl status kubelet
```

## ✅ Verify Cluster

```bash
kubectl get nodes
kubectl get pods -A
kubectl cluster-info
```

Expected:
Node status: Ready
All system pods: Running

## 🌐 Kubernetes Dashboard (Local Access)

```bash
# Install dashboard
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

# Start proxy (keep terminal open)
kubectl proxy

# Open Dashboard URL
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

# Login Token
kubectl -n kubernetes-dashboard create token admin-user

# If admin not found (Create ServiceAccount)
kubectl create serviceaccount admin-user -n kubernetes-dashboard

# Give admin permissions
kubectl create clusterrolebinding admin-user \
  --clusterrole=cluster-admin \
  --serviceaccount=kubernetes-dashboard:admin-user

# Generate token
kubectl -n kubernetes-dashboard create token admin-user

```

## 🧭 Dashboard Navigation

- Overview – Cluster summary
- Workloads
  - Pods
  - Deployments
  - ReplicaSets
- Services
- Config
- Cluster

## 🚀 First Application: NGINX Pod

```bash
# Create Pod
kubectl run my-nginx --image=nginx --port=80

# Verify
kubectl get pods
kubectl describe pod my-nginx

```

## 🌐 Expose Pod (Networking)

```bash
# Pods are internal. We need a Service.
kubectl expose pod my-nginx --type=NodePort --port=80

# Check service:
kubectl get svc

# Access in browser:
http://<NODE-IP>:<NODE-PORT>

```

## 🧹 Delete Pod & Service

```bash
kubectl delete pod my-nginx
kubectl delete svc my-nginx
```

## 🚀 Real Way: Deployment (Docker Hub Image)

Pods are temporary. Deployments are production-ready.

```bash
# Create Deployment
kubectl create deployment nginx-deploy --image=nginx

# Verify
kubectl get deployments
kubectl get pods

```

## 🌐 Expose Deployment

```bash
kubectl expose deployment nginx-deploy --type=NodePort --port=80
kubectl get svc
http://<NODE-IP>:<NODE-PORT>

```

## 🔗 Pod Networking Basics

```bash
# Pod IPs
kubectl get pods -o wide

# Each Pod has its own IP
# Pods communicate directly via Calico CNI

| Service Type | Purpose            |
| ------------ | ------------------ |
| ClusterIP    | Internal only      |
| NodePort     | Access via Node IP |
| LoadBalancer | Cloud only         |

```

## 🧪 Test Inside Pod

```bash
kubectl exec -it <pod-name> -- bash

# Inside pod:
curl localhost

```

## 📦 Namespaces

```bash
kubectl get ns
kubectl create ns dev
kubectl get pods -n dev
```

## 📜 YAML Commands

```bash
kubectl apply -f app.yaml
kubectl delete -f app.yaml
kubectl get pod my-pod -o yaml
```

## 🔧 Debugging Commands

```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl get events
```

## 🧹 Cleanup (IMPORTANT)

```bash
kubectl delete deployment nginx-deploy
kubectl delete svc nginx-deploy
kubectl delete all --all
```

## 🧠 Golden kubectl Commands (Must Remember)

```bash
kubectl get pods
kubectl describe pod
kubectl logs
kubectl exec -it
kubectl apply -f
kubectl delete
```

