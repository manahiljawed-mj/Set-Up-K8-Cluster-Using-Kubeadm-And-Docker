# 🚀 Kubernetes Cluster Setup with Kubeadm

Welcome to setting up your Kubernetes cluster using Kubeadm! This guide is inspired by and adapted from 
[MrDevSecOps' Medium article](https://medium.com/@mrdevsecops/set-up-a-kubernetes-cluster-with-kubeadm-508db74028ce) and 
[Kubernetes Documentation](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/). 
It will walk you through the necessary steps to get your cluster up and running.

## Prerequisites

Before you begin, ensure you have the following:

- **Linux Distribution**: Ubuntu 18.04 or later (or your preferred distribution)
- **Root Access**: You'll need `sudo` or root access for installation and configuration.
- **Internet Access**: To download necessary packages and container images.

## Step-by-Step Setup

### 1. Disable Swap

```bash
sudo su
```
```bash
swapoff -a
```
```bash
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```
ℹ️ Disabling swap ensures Kubernetes can manage memory efficiently.

### 2. Enable Netfilter Modules

```bash
lsmod | grep br_netfilter
```
```bash
sudo modprobe br_netfilter
```
```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```
```bash
sudo sysctl --system
```
🔧 Ensures proper IP forwarding and iptables handling required by Kubernetes networking.

### 3. Install Docker

```bash
apt-get update
```
```bash
apt-get install -y docker.io
```
```bash
systemctl start docker
```
🐳 Docker is used as the container runtime for Kubernetes.

### 4. Configure Docker

```bash
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
```
```bash
systemctl daemon-reload
```
```bash
systemctl restart docker
```
🔧 Configures Docker to use systemd as the cgroup driver and sets logging options.

### 5. Install Kubernetes Components

```bash
sudo apt-get update
```
```bash
sudo apt-get install -y apt-transport-https ca-certificates curl
```
```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
```bash
apt-get update
```
```bash
apt-get install -y kubelet kubeadm kubectl
```
```bash
apt-mark hold kubelet kubeadm kubectl
```
###
☸️ Installs Kubernetes components: kubelet, kubeadm, and kubectl.

### 6. Initialize Kubernetes Master Node
```bash
kubeadm init --pod-network-cidr=192.168.0.0/16
```
🎯 Initializes the Kubernetes control plane on the master node.

### 7.Configure Kubectl

```bash
mkdir -p $HOME/.kube
```
```bash
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```
```bash
chown $(id -u):$(id -g) $HOME/.kube/config
```
🔧 Configures kubectl to communicate with the Kubernetes cluster.

### 8. Install Pod Network Addon
```bash
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```
🌐 Installs the Weave Net plugin to enable networking between pods.

###  9. Join Worker Nodes (Optional)
```bash
kubeadm join 10.128.0.9:6443 --token 21dg74.rkaqfcksuut150xm \
    --discovery-token-ca-cert-hash sha256:69cffbab4446f21047711bd074074747daa4211508c973931c0c7f177db4f108
```
🔗 Command to join worker nodes to the Kubernetes cluster.

### Additional Commands
For managing tokens and nodes:

```bash
kubeadm token create —-print-join-command
```
```bash
kubectl get nodes -o wide
```
📝 Generates a join token and verifies node status.

🎉 Congratulations! You've successfully set up a Kubernetes cluster using kubeadm. You can now deploy and manage your applications using Kubernetes.

For more details and troubleshooting tips, refer to Kubernetes Documentation.



