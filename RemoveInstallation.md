# Kubernetes Full Cleanup Script (Dashboard + Helm + Cluster)

This script **removes Kubernetes cluster, Dashboard, Helm, container runtimes, networking, and all configs** for a fresh reinstall.

```bash
#!/bin/bash

echo "=== 1️⃣ Remove Kubernetes Dashboard ==="
kubectl delete namespace kubernetes-dashboard --ignore-not-found
kubectl delete clusterrolebinding admin-user --ignore-not-found
kubectl delete serviceaccount admin-user -n kubernetes-dashboard --ignore-not-found

echo "=== 2️⃣ Remove Helm ==="
# Remove helm binary if installed via curl script
sudo rm -f /usr/local/bin/helm

# Remove snap version if installed
sudo snap remove helm --classic

echo "=== 3️⃣ Reset kubeadm Cluster ==="
sudo kubeadm reset -f

echo "=== 4️⃣ Remove Kubernetes Packages ==="
sudo apt-get purge -y kubeadm kubectl kubelet
sudo apt-get autoremove -y

echo "=== 5️⃣ Check if binaries still exist ==="
which kubectl || echo "kubectl removed"
which kubeadm || echo "kubeadm removed"
which kubelet || echo "kubelet removed"
which helm || echo "helm removed"

echo "=== 6️⃣ Remove CNI Plugins and Networking ==="
sudo rm -rf /etc/cni/net.d
sudo rm -rf /opt/cni/bin
sudo ip link delete cni0 2>/dev/null
sudo ip link delete flannel.1 2>/dev/null
sudo ip link delete cali* 2>/dev/null
sudo ip link delete docker0 2>/dev/null

echo "Flush iptables"
sudo iptables -F
sudo iptables -t nat -F
sudo iptables -t mangle -F
sudo iptables -X

echo "=== 7️⃣ Remove Cluster Configs and Certificates ==="
sudo rm -rf ~/.kube
sudo rm -rf /etc/kubernetes
sudo rm -rf /var/lib/etcd
sudo rm -rf /var/lib/kubelet
sudo rm -rf /var/lib/dockershim
sudo rm -rf /var/run/kubernetes

echo "=== 8️⃣ Remove containerd / Docker (Optional) ==="
sudo systemctl stop containerd
sudo systemctl disable containerd
sudo apt-get purge -y containerd docker docker.io docker-ce docker-ce-cli
sudo apt-get autoremove -y
sudo rm -rf /etc/containerd
sudo rm -rf /var/lib/containerd
sudo rm -rf /var/lib/docker

echo "=== 9️⃣ Verify System is Clean ==="
kubectl version || echo "kubectl not found"
kubeadm version || echo "kubeadm not found"
systemctl status kubelet || echo "kubelet removed"
containerd --version || echo "containerd removed"
helm version || echo "helm removed"

echo "✅ Cleanup Complete. Optional: Reboot for full refresh"
# sudo reboot
