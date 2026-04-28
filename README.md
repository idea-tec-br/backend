# Backend

## Nuvem Kubernetes

Sistema operacional: Debian 13 (trixie).

### Instalação

CRI ([fonte](https://kubernetes.io/docs/setup/production-environment/container-runtimes/)):

```sh
apt -y install containerd

cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.ipv4.conf.all.forwarding = 1
net.ipv6.conf.all.forwarding = 1
EOF
sysctl --system
```

Ferramentas ([fonte](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)):

```sh
apt install -y apt-transport-https ca-certificates curl gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.36/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.36/deb/ /' | tee /etc/apt/sources.list.d/kubernetes.list
apt -y modernize-sources
apt update
apt -y install kubelet kubeadm kubectl
systemctl enable --now kubelet
```

Cluster ([fonte](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)):

```sh
# ...
```
