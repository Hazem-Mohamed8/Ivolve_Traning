# Lab 9: Kubernetes Cluster with kubeadm on AWS

## Goal

Build a 2-node Kubernetes cluster (1 master + 1 worker) with kubeadm, using containerd and the Calico CNI.

## Setup

- AWS EC2, 2 × `t3.small`, Amazon Linux 2023
- Kubernetes v1.37.1, containerd 2.3.7, Calico v3.33.0
- Security group: SSH from my IP, all traffic inside the group

## Steps

**1. On both nodes: prerequisites and containerd**

```bash
swapoff -a
modprobe overlay && modprobe br_netfilter
sysctl -w net.ipv4.ip_forward=1 net.bridge.bridge-nf-call-iptables=1
setenforce 0

dnf install -y containerd iproute-tc conntrack-tools
containerd config default > /etc/containerd/config.toml
sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
systemctl enable --now containerd
```

**2. On both nodes: install kubeadm, kubelet, kubectl** (Kubernetes repo `pkgs.k8s.io`, stable v1.37)

```bash
dnf install -y kubelet kubeadm kubectl --setopt=disable_excludes=kubernetes
systemctl enable --now kubelet
```

**3. Master: initialize the control plane**

```bash
kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=172.31.13.111
```

**4. Master: configure kubectl**

```bash
mkdir -p ~/.kube
sudo cp /etc/kubernetes/admin.conf ~/.kube/config
sudo chown $(id -u):$(id -g) ~/.kube/config
```

**5. Master: install Calico**

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.33.0/manifests/tigera-operator.yaml
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.33.0/manifests/custom-resources.yaml
```

**6. Worker: join the cluster** (command printed by `kubeadm token create --print-join-command` on the master)

```bash
kubeadm join 172.31.13.111:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

## Verification

```bash
kubectl get nodes -o wide
kubectl get pods -A
```
Test deployment (nginx on the worker):

```bash
kubectl create deployment nginx --image=nginx --replicas=2
kubectl expose deployment nginx --port=80 --type=NodePort
kubectl get pods -o wide
```
