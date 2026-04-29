# Backend

## Nuvem Kubernetes

- Sistema operacional: Ubuntu 26.04 LTS.

- Atualização do sistema:

```sh
echo 'Acquire::ForceIPv4 "true";' | sudo tee /etc/apt/apt.conf.d/99force-ipv4 > /dev/null
sudo apt update
sudo apt upgrade -y
sudo apt dist-upgrade -y
sudo do-release-upgrade
sudo apt autopurge
sudo apt clean
sudo reboot
```

### Instalação

- [CRI](https://kubernetes.io/docs/setup/production-environment/container-runtimes/):

```sh
sudo apt install -y containerd
sudo install -d /etc/containerd
containerd config default | sed 's/SystemdCgroup = false/SystemdCgroup = true/g' |sudo tee /etc/containerd/config.toml > /dev/null
sudo systemctl restart containerd

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf > /dev/null
net.ipv4.conf.all.forwarding = 1
net.ipv6.conf.all.forwarding = 1
EOF
sudo sysctl --system
```

- [Ferramentas](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/):

```sh
sudo apt install -y apt-transport-https ca-certificates curl gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.36/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.36/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo systemctl enable --now kubelet
```

- [_Cluster_](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/) com suporte a [pilha dupla](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/dual-stack-support/):

```sh
kubeadm init --v=5 --control-plane-endpoint=k8s-0.sj.ifsc.edu.br --pod-network-cidr=10.0.0.0/16,fc00::/56 --service-cidr=10.1.0.0/16,fc00:1::/112
```

- Ativação dos nós (_workers_):

```sh
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```
