# Backend

## Nuvem Kubernetes

- Sistema operacional: Ubuntu 26.04 LTS.

- Atualização do sistema:

```sh
# Preferência por IPv4
echo 'Acquire::ForceIPv4 "true";' | sudo tee /etc/apt/apt.conf.d/99force-ipv4 > /dev/null
#
# Atualização
sudo apt update
sudo apt upgrade -y
sudo apt dist-upgrade -y
sudo do-release-upgrade
#
# Limpeza
sudo apt autopurge
sudo apt clean
#
# Reinício
sudo reboot
```

### Instalação

- [CRI](https://kubernetes.io/docs/setup/production-environment/container-runtimes/):

```sh
# Instalação
sudo apt install -y containerd
#
# Configuração
sudo install -d /etc/containerd
containerd config default | sed 's/SystemdCgroup = false/SystemdCgroup = true/g' |sudo tee /etc/containerd/config.toml > /dev/null
sudo systemctl restart containerd
#
# Roteamento IPv4 e IPv6
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf > /dev/null
net.ipv4.conf.all.forwarding = 1
net.ipv6.conf.all.forwarding = 1
EOF
sudo sysctl --system
```

- [Ferramentas](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/):

```sh
# Instalação
sudo apt install -y apt-transport-https ca-certificates curl gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.36/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.36/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo systemctl enable --now kubelet
```

- [_Cluster_](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/) com suporte a [pilha dupla](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/dual-stack-support/) e [alta disponibilidade](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/):

```sh
# Criação do cluster
sudo kubeadm init --v=5 --control-plane-endpoint=k8s.sj.ifsc.edu.br --pod-network-cidr=10.0.0.0/16,fc00::/56 --service-cidr=10.1.0.0/16,fc00:1::/112 --upload-certs
```

As ações a serem feitas nos outros _control planes_ serão informadas no final desse comando.

- Ativação dos nós (_control planes_ como _workers_):

```sh
# Ativação dos control planes
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

- Instalação de [_add-on_ de rede (CNI)](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#pod-network) [Cilium](https://docs.cilium.io/en/stable/gettingstarted/k8s-install-default/) com [Hubble](https://docs.cilium.io/en/stable/observability/hubble/setup/#hubble-setup) ativado:

```sh
# Instalação do Cilium CLI
CILIUM_CLI_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/cilium-cli/main/stable.txt)
CLI_ARCH=amd64
if [ "$(uname -m)" = "aarch64" ]; then CLI_ARCH=arm64; fi
curl -L --fail --remote-name-all https://github.com/cilium/cilium-cli/releases/download/${CILIUM_CLI_VERSION}/cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
sha256sum --check cilium-linux-${CLI_ARCH}.tar.gz.sha256sum
sudo tar xzvfC cilium-linux-${CLI_ARCH}.tar.gz /usr/local/bin
rm cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
#
# Instalação do Hubble CLI
HUBBLE_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/hubble/main/stable.txt)
HUBBLE_ARCH=amd64
if [ "$(uname -m)" = "aarch64" ]; then HUBBLE_ARCH=arm64; fi
curl -L --fail --remote-name-all https://github.com/cilium/hubble/releases/download/$HUBBLE_VERSION/hubble-linux-${HUBBLE_ARCH}.tar.gz{,.sha256sum}
sha256sum --check hubble-linux-${HUBBLE_ARCH}.tar.gz.sha256sum
sudo tar xzvfC hubble-linux-${HUBBLE_ARCH}.tar.gz /usr/local/bin
rm hubble-linux-${HUBBLE_ARCH}.tar.gz{,.sha256sum}
#
# Instalação do Cilium
cilium install --version 1.19.3
cilium hubble enable
#
# Verificação
cilium status --wait
hubble status -P
hubble observe -P
#
# Teste
cilium connectivity test
```
