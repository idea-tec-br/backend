# 01: Rede

Escolhida a pilha dupla IPv6 (preferencial) e IPv4 tanto nas máquinas físicas quanto na rede virtualizada via [CNI](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/).

## Configuração

Executar em todas as máquinas:

```sh
# Módulos do kernel
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter
#
# Parâmetros do kernel
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
net.ipv6.conf.all.forwarding        = 1
net.ipv6.conf.default.forwarding    = 1
EOF
sudo sysctl --system
#
# DNS IPv6 + IPv4
cat <<EOF | sudo tee /etc/k8s-resolv.conf
nameserver 2606:4700:4700::1111
nameserver 2606:4700:4700::1001
nameserver 1.1.1.1
EOF
#
# TCP offloading desativado
echo "      transmit-checksum-offload: false" | sudo tee -a /etc/netplan/50-cloud-init.yaml
echo "      receive-checksum-offload: false" | sudo tee -a /etc/netplan/50-cloud-init.yaml
sudo netplan apply
```
