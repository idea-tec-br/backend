# Backend

## Nuvem Kubernetes

Sistema operacional: Ubuntu 26.04 LTS.

Atualização do sistema:

```sh
echo 'Acquire::ForceIPv4 "true";' | sudo tee /etc/apt/apt.conf.d/99force-ipv4
sudo apt update
sudo apt upgrade -y
sudo apt dist-upgrade -y
sudo do-release-upgrade -d
sudo apt --purge autoremove
sudo reboot
```

### Instalação

CRI ([fonte](https://kubernetes.io/docs/setup/production-environment/container-runtimes/)):

```sh
sudo apt install -y containerd

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.conf.all.forwarding = 1
net.ipv6.conf.all.forwarding = 1
EOF
sudo sysctl --system
```

Ferramentas ([fonte](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)):

```sh
sudo apt install -y apt-transport-https ca-certificates curl gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.36/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.36/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo systemctl enable --now kubelet
```

Cluster ([fonte](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)):

```sh
# ...
```
