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
echo "      transmit-checksum-offload: false" | sudo tee -a /etc/netplan/50-cloud-init.yaml
echo "      receive-checksum-offload: false" | sudo tee -a /etc/netplan/50-cloud-init.yaml
sudo netplan apply
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
sudo kubeadm init --config=k8s-0.yaml --upload-certs
```

Ns demais máquinas, deve-se atualizar o arquivo do `kubeadm` com os valores informados pelo comando anterior e depois rodar na segunda máquina:

```sh
sudo kubeadm join --config=k8s-1.yaml
```

e na terceira máquina: 

```sh
sudo kubeadm join --config=k8s-2.yaml
```

Apenas na primeira máquina:

```sh
kubectl taint nodes --all node-role.kubernetes.io/control-plane:NoSchedule-
```

### Helm

Em todas as máquinas:

```sh
HELM_BUILDKITE_APT_KEY_ID="DDF78C3E6EBB2D2CC223C95C62BA89D07698DBC6"
sudo apt-get install curl gpg apt-transport-https --yes
curl -fsSL https://packages.buildkite.com/helm-linux/helm-debian/gpgkey > "${TMPDIR:-/tmp}/helm.gpg"
if [ "$(gpg --show-keys --with-colons "${TMPDIR:-/tmp}/helm.gpg" | awk -F: '$1 == "fpr" {print $10}' | head -n 1)" != "${HELM_BUILDKITE_APT_KEY_ID}" ]; then echo "ERROR: Unexpected Helm APT key ID: potential key compromise"; exit 1; fi
cat "${TMPDIR:-/tmp}/helm.gpg" | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/helm.gpg] https://packages.buildkite.com/helm-linux/helm-debian/any/ any main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

### Gateway API

Apenas na primeira máquina:

```sh
kubectl apply --server-side -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.5.1/experimental-install.yaml
```

### Instalação de [_add-on_ de rede (CNI)](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#pod-network) [Flannel](https://github.com/flannel-io/flannel) com suporte a [pilha dupla](https://github.com/flannel-io/flannel/blob/master/Documentation/configuration.md)

Apenas na primeira máquina:

```sh
kubectl apply -f kube-flannel.yaml
```

### Criação do certificado digital único

Apenas na primeira máquina:

```sh
sudo apt install certbot
sudo certbot certonly --manual --preferred-challenges dns --email etorresini@ifsc.edu.br --agree-tos -d '*.k8s.sj.ifsc.edu.br' -d 'k8s.sj.ifsc.edu.br'
```

No processo, será criado um registro DNS para ser adicionado no servidor, o que permitirá criar o certificado digital. Uma vez criado, rodar na primeira máquina:

```sh
kubectl create secret tls wildcard-k8s-ifsc-tls \
  --cert=/etc/letsencrypt/live/k8s.sj.ifsc.edu.br/fullchain.pem \
  --key=/etc/letsencrypt/live/k8s.sj.ifsc.edu.br/privkey.pem \
  -n default
```

### SDS: [Longhorn](https://longhorn.io/)

Todas as máquinas:

```sh
sudo apt install -y open-iscsi nfs-common util-linux cryptsetup dmsetup
sudo systemctl enable --now iscsid
sudo modprobe iscsi_tcp
echo "iscsi_tcp" | sudo tee /etc/modules-load.d/iscsi.conf
```

Apenas a primeira máquina:

```sh
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/v1.12.0/deploy/longhorn.yaml
```

Em todas as máquinas:

```sh
curl -sSfL -o longhornctl https://github.com/longhorn/cli/releases/download/v1.12.0/longhornctl-linux-amd64
chmod 755 longhornctl 
sudo mv longhornctl /usr/local/bin
```

### Acesso

Apenas na primeira máquina:

```sh
# k9s
curl -LO https://github.com/derailed/k9s/releases/download/v0.51.0/k9s_linux_amd64.deb
sudo dpkg -i k9s_linux_amd64.deb
rm -f k9s_linux_amd64.deb
#
# Headlamp
helm repo add headlamp https://kubernetes-sigs.github.io/headlamp/
helm install \
  headlamp headlamp/headlamp \
  --namespace headlamp \
  --create-namespace
kubectl create token headlamp -n headlamp # Salvar para uso na interface Web
kubectl port-forward -n headlamp service/headlamp 8080:80
```

### Teste

Apenas na primeira máquina:

```sh
kubectl apply -f teste.yaml
```

Com isso, deverão estar disponíveis os endereços `app1.k8s.sj.ifsc.edu.br` e `app2.k8s.sj.ifsc.edu.br` no navegador.
