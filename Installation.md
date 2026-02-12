# 🚀 Kubernetes Setup on Ubuntu 25 (Beginner → Pro)

This guide helps you **install, configure, and verify Kubernetes on Ubuntu 25** using **kubeadm**, following **official Kubernetes best practices**.  
It also includes **API references** and **learning checkpoints**.

---

## 📌 What You Will Install

- **containerd** (Container Runtime)
- **kubeadm** (Cluster bootstrap tool)
- **kubelet** (Node agent)
- **kubectl** (CLI tool)
- **CNI plugin** (Networking – Calico)

---

## 🧠 Prerequisites

### System Requirements

- Ubuntu **25.x**
- 2 CPU cores (minimum)
- 2 GB RAM (minimum, 4 GB recommended)
- Swap **disabled**
- Root or sudo access
- Internet connectivity

---

## 🧹 Step 1: Prepare the System

### Update OS

```bash
sudo apt update && sudo apt upgrade -y
```

### Disable Swap (Required)

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

### Enable Required Kernel Modules

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

### Set sysctl Parameters

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
net.ipv4.ip_forward=1
EOF

sudo sysctl --system
```

## 🐳 Step 2: Install containerd (Container Runtime)

### Install containerd

```bash
sudo apt install -y containerd
```

### Configure containerd

```bash
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml

```

### Enable Systemd Cgroup Driver (IMPORTANT)

```bash
# Edit
sudo nano /etc/containerd/config.toml

# Set
SystemdCgroup = true

#Whwre need to set (search this in nano)
# [plugins.'io.containerd.cri.v1.runtime'.containerd.runtimes.runc.options]
#   BinaryName = ''
#   CriuImagePath = ''
#   CriuWorkPath = ''
#   IoGid = 0
#   IoUid = 0
#   NoNewKeyring = false
#   Root = ''
#   ShimCgroup = ''

# After set
# [plugins.'io.containerd.cri.v1.runtime'.containerd.runtimes.runc.options]
#   BinaryName = ''
#   CriuImagePath = ''
#   CriuWorkPath = ''
#   IoGid = 0
#   IoUid = 0
#   NoNewKeyring = false
#   Root = ''
#   ShimCgroup = ''
#   SystemdCgroup = true

# Run this
sudo systemctl restart containerd
sudo systemctl enable containerd

```

## ☸ Step 3: Install Kubernetes Components

### Add Kubernetes Repository

```bash
sudo apt-get update
# apt-transport-https may be a dummy package; if so, you can skip that package
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg

# If the folder `/etc/apt/keyrings` does not exist, it should be created before the curl command, read the note below.
# sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.35/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg # allow unprivileged APT programs to read this keyring
# This overwrites any existing configuration in /etc/apt/sources.list.d/kubernetes.list
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.35/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo chmod 644 /etc/apt/sources.list.d/kubernetes.list   # helps tools such as command-not-found to work correctly
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

```

## 🧠 Step 4: Initialize Kubernetes Cluster (Master Node)

### Why kubeadm init is required

- Bootstraps Kubernetes control plane
- Generates TLS certificates
- Creates kubeconfig files
- Defines pod network CIDR for CNI
- Outputs secure join command for worker nodes

🎯 Final answer in one line

We run kubeadm init to create the Kubernetes control plane and define the pod network, without which Kubernetes cannot function.

```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```

## 🧩 Step 5: Configure kubectl Access

1️⃣ Why do we do this?

kubectl is the command-line tool for Kubernetes. It needs to know where the cluster is and how to authenticate.

The admin.conf file contains:

API server address (where Kubernetes API lives)

User credentials & certificates (for secure access)

Cluster info (name, context, namespace)

Without this file, kubectl would have no idea where the cluster is or how to talk to it.

```bash
# Makes a hidden directory in your home to store kubeconfig
# kubectl looks here by default
mkdir -p $HOME/.kube

# Moves the cluster access config to your home directory
# Now your user can use kubectl, not just root
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config

# Ensures your user (not root) owns the file
# Prevents permission errors when running kubectl
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Verify:
kubectl get nodes

# Response will be like
NAME       STATUS     ROLES           AGE     VERSION
stpl1017   NotReady   control-plane   3m21s   v1.35.0

```

## 🌐 Step 6: Install Network Plugin (Calico)

**Goal:** Enable Pod-to-Pod and Pod-to-Service communication across the cluster.

After initializing the cluster, **Kubernetes has no networking** configured. Pods won’t be able to communicate until a network plugin is installed. Calico is one of the most popular CNI plugins that provides networking and network security policies.

### Why we need a network plugin

- Kubernetes does **not come with built-in Pod networking**.  
- A CNI plugin like **Calico** provides:
  - Pod-to-Pod networking across nodes
  - Pod-to-Service communication
  - Network policies (firewall rules between pods)
- Without a plugin, pods remain in **Pending state** because they cannot get an IP.

### Install Calico

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

# Verify Installation
kubectl get pods -n kube-system

```

**The manifest sets up:**

- Calico controllers
- Calico DaemonSets on each node
- ConfigMaps and RBAC for Calico
- It automatically configures the cluster network.

## 🧑‍💻 Step 7: Allow Pods on Control Plane (Single Node Lab)

By default, Kubernetes taints the control-plane node so that no regular pods are scheduled on it.
In a single-node cluster (like labs or testing), we need to allow pods to run on the control-plane node.

```bash
# Remove taint
kubectl taint nodes --all node-role.kubernetes.io/control-plane-

# Output should be untainted
```

### Why & When:

**Why:** Control-plane nodes are tainted to prevent production workloads from affecting cluster management.

**When:** Only remove the taint in single-node or test clusters. In production, keep control-plane nodes dedicated to management.

**Effect:** Now you can deploy normal applications (pods, deployments) on your single-node cluster.

## ✅ Step 8: Verify Cluster

```bash
kubectl get nodes -o wide
kubectl get pods -A
```

## 📦 Kubernetes API Reference

```table
| API Group                    | Purpose                    |
| ---------------------------- | -------------------------- |
| core/v1                      | Pods, Services, ConfigMaps |
| apps/v1                      | Deployments, StatefulSets  |
| batch/v1                     | Jobs, CronJobs             |
| networking.k8s.io/v1         | Ingress, NetworkPolicy     |
| rbac.authorization.k8s.io/v1 | Roles, RBAC                |
```

## 🛠 Common Debug Commands

```bash
kubectl describe pod <pod>
kubectl logs <pod>
kubectl exec -it <pod> -- /bin/bash
kubectl get events
```

## Verify kubectl is Configured Correctly

```bash
kubectl version --client
kubectl cluster-info
```

**What to see:**

- Client version shows your kubectl version
- Cluster info shows API server URL and Kubernetes dashboard (if installed)
- No connection errors

**check Cluster Nodes**

```bash
kubectl get nodes -o wide
```

**Check System Pods (Control Plane Components)**

```bash
kubectl get pods -n kube-system

# See the pods log
kubectl logs <pod-name> -n kube-system
```

**Test kubelet and Node Agents**

```bash
systemctl status kubelet
```

---

## Services That Must Start After Reboot

```table
| Service                                  | Purpose                      | Action after reboot                        |
| ---------------------------------------- | ---------------------------- | ------------------------------------------ |
| `kubelet`                                | Node agent that manages pods | Should start automatically if enabled      |
| `containerd`                             | Container runtime            | Should start automatically if enabled      |
| `docker` (if used)                       | Container runtime            | Should start automatically if enabled      |
| Network plugins (Calico / Flannel / CNI) | Pod networking               | No manual start needed; handled by kubelet |

```

```bash
sudo systemctl status kubelet
sudo systemctl status containerd

# Enable them if not:
sudo systemctl enable kubelet
sudo systemctl enable containerd

```

**Verify Cluster After Reboot**

```bash
# Check nodes
kubectl get nodes

# Check system pods
kubectl get pods -n kube-system

# Check workloads
kubectl get pods -A
```

### Network & Pod Connectivity Check

```bash
# Pod stuck in Pending → Check node resources:
kubectl describe pod <pod-name>

# Network plugin failed → Reapply CNI plugin:
kubectl apply -f <cni-plugin.yaml>

```

### Persistent Storage Check

```bash
kubectl get pvc
kubectl get pv
kubectl describe pvc <pvc-name>
```

### Best Practices After Reboot

```bash
# Enable kubelet & containerd services
sudo systemctl enable kubelet containerd

# Check cluster health
kubectl get nodes
kubectl get pods -A

# Check events
kubectl get events --sort-by=.metadata.creationTimestamp

# Restart failed pods if necessary
kubectl delete pod <pod-name>

```

## Install Kubernetes Dashboard
