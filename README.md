# Backend

## Nuvem Kubernetes

Sistema operacional: Ubuntu 26.04 LTS.

### Atualização do sistema

Todas as máquinas:

```sh
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

### Rede

Todas as máquinas:

```sh
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter
#
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
net.ipv6.conf.all.forwarding        = 1
net.ipv6.conf.default.forwarding    = 1
EOF
sudo sysctl --system
```

### Instalação

[CRI](https://kubernetes.io/docs/setup/production-environment/container-runtimes/):

Todas as máquinas:

```sh
# Instalação
sudo apt install -y containerd
#
# Configuração
sudo install -d /etc/containerd
containerd config default | sed 's/SystemdCgroup = false/SystemdCgroup = true/g' |sudo tee /etc/containerd/config.toml > /dev/null
sudo systemctl restart containerd
```

### [Ferramentas](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

Todas as máquinas:

```sh
# Instalação
sudo apt install -y apt-transport-https ca-certificates curl gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.36/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.36/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt install -y kubeadm kubectl
```

### Kubelet

Todas as máquinas:

```sh
sudo apt install -y kubelet
echo "KUBELET_EXTRA_ARGS=\"--node-ip=`hostname -I|sed  's/\ /,/'|xargs`\"" | sudo tee /etc/default/kubelet
sudo systemctl enable --now kubelet
sudo systemctl restart kubelet
```


### [_Cluster_](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/) com suporte a [pilha dupla](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/dual-stack-support/) e [alta disponibilidade](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/)

Apenas na primeira máquina:

```sh
# Criação do cluster
sudo kubeadm init --v=5 --config=kubeadm-config.yaml
```

As ações a serem feitas nos outros _control planes_ serão informadas no final desse comando.

### Ativação dos nós (_control planes_ como _workers_)

Apenas na primeira máquina:

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
cilium install \
  --set ipv4.enabled=true \
  --set ipv6.enabled=true \
  --set ipam.mode=kubernetes \
  --set ipam.operator.clusterPoolIPv4PodCIDRList="10.0.0.0/16" \
  --set ipam.operator.clusterPoolIPv6PodCIDRList="fc00::/56" \
  --set ipv4NativeRoutingCIDR="10.1.0.0/16" \
  --set ipv6NativeRoutingCIDR="fd00::/112" \
  --set kubeProxyReplacement=true
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

### Instalação do [Longhorn](https://longhorn.io/) para armazenamento

Todas as máquinas:

```sh
sudo apt install -y open-iscsi nfs-common util-linux
sudo systemctl enable --now iscsid

sudo modprobe iscsi_tcp
echo "iscsi_tcp" | sudo tee /etc/modules-load.d/iscsi.conf
```

* Instalação do API Gateway

```sh
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/latest/download/standard-install.yaml
kubectl -n longhorn-system port-forward svc/longhorn-frontend 8080:80
xdg-open http://localhost:8080
```

