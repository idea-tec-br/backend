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

### Preparação da rede

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
#
cat <<EOF | sudo tee /etc/k8s-resolv.conf
nameserver 2606:4700:4700::1111
nameserver 2606:4700:4700::1001
nameserver 1.1.1.1
EOF
# 
sudo apt-get install -y ethtool
sudo ethtool -K eth0 tx off rx off
```

### CRI: containerd

[CRI](https://kubernetes.io/docs/setup/production-environment/container-runtimes/):

Todas as máquinas:

```sh
sudo apt install -y containerd
sudo install -d /etc/containerd
containerd config default | sed 's/SystemdCgroup = false/SystemdCgroup = true/g' |sudo tee /etc/containerd/config.toml > /dev/null
sudo systemctl restart containerd
```

### [Ferramentas](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

Todas as máquinas:

```sh
sudo apt install -y apt-transport-https ca-certificates curl gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.36/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.36/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt install -y kubeadm kubectl kubelet
sudo systemctl enable --now kubelet
sudo systemctl restart kubelet
```

### [_Cluster_](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/) com suporte a [pilha dupla](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/dual-stack-support/)

Apenas na primeira máquina:

```sh
sudo kubeadm init --v=5 --config=kubeadm-config.yaml
kubectl taint nodes k8s-0.sj.ifsc.edu.br node-role.kubernetes.io/control-plane:NoSchedule-
```

Para as demais máquinas, deve-se atualizar o arquivo do `kubeadm` com os valores informados pelo comando anterior e depois rodar:

```sh
sudo kubeadm join --v=5 --config=kubeadm-config.yaml
```

### Gateway API

Apenas na primeira máquina:

```sh
kubectl apply --server-side -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.5.1/experimental-install.yaml
```

### SDN: [_add-on_ de rede (CNI)](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#pod-network) [Cilium](https://docs.cilium.io/en/stable/gettingstarted/k8s-install-default/) com [Hubble](https://docs.cilium.io/en/stable/observability/hubble/setup/#hubble-setup)

Apenas na primeira máquina:

```sh
CILIUM_CLI_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/cilium-cli/main/stable.txt)
CLI_ARCH=amd64
if [ "$(uname -m)" = "aarch64" ]; then CLI_ARCH=arm64; fi
curl -L --fail --remote-name-all https://github.com/cilium/cilium-cli/releases/download/${CILIUM_CLI_VERSION}/cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
sha256sum --check cilium-linux-${CLI_ARCH}.tar.gz.sha256sum
sudo tar xzvfC cilium-linux-${CLI_ARCH}.tar.gz /usr/local/bin
rm cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
#
HUBBLE_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/hubble/main/stable.txt)
HUBBLE_ARCH=amd64
if [ "$(uname -m)" = "aarch64" ]; then HUBBLE_ARCH=arm64; fi
curl -L --fail --remote-name-all https://github.com/cilium/hubble/releases/download/$HUBBLE_VERSION/hubble-linux-${HUBBLE_ARCH}.tar.gz{,.sha256sum}
sha256sum --check hubble-linux-${HUBBLE_ARCH}.tar.gz.sha256sum
sudo tar xzvfC hubble-linux-${HUBBLE_ARCH}.tar.gz /usr/local/bin
rm hubble-linux-${HUBBLE_ARCH}.tar.gz{,.sha256sum}
#
cilium install \
  --set ipv6.enabled=true \
  --set ipv4.enabled=true \
  --set kubeProxyReplacement=true \
  --set gatewayAPI.enabled=true \
  --set ipam.mode=cluster-pool \
  --set ipam.operator.clusterPoolIPv6PodCIDRList='{fc00::/48}' \
  --set ipam.operator.clusterPoolIPv6MaskSize=64 \
  --set ipam.operator.clusterPoolIPv4PodCIDRList='{10.0.0.0/16}' \
  --set ipam.operator.clusterPoolIPv4MaskSize=24
cilium hubble enable
#
cilium status --wait
hubble status -P
hubble observe -P
#
cilium connectivity test
```

### SDS: [Longhorn](https://longhorn.io/)

Todas as máquinas:

```sh
sudo apt install -y open-iscsi nfs-common util-linux
sudo systemctl enable --now iscsid
sudo modprobe iscsi_tcp
echo "iscsi_tcp" | sudo tee /etc/modules-load.d/iscsi.conf
```

### K9s

```sh
curl -LO https://github.com/derailed/k9s/releases/download/v0.51.0/k9s_linux_amd64.deb
sudo dpkg -i k9s_linux_amd64.deb
rm -f k9s_linux_amd64.deb
```