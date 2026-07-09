# 04: _Cluster_

Escolhido o [_cluster_](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/) com [pilha dupla](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/dual-stack-support/) e [HA](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/ha-topology/).

## Instalação

Executar na primeira máquina, `k8s-0`:

```sh
# init
sudo kubeadm init --config=k8s-0.yaml --upload-certs
```

Ns demais máquinas, deve-se atualizar o arquivo do `kubeadm` com os valores informados pelo comando anterior.

Executar na segunda máquina, `k8s-1`:

```sh
# join
sudo kubeadm join --config=k8s-1.yaml
```

Executar na terceira máquina, `k8s-2`: 

```sh
# join
sudo kubeadm join --config=k8s-2.yaml
```

## Configuração do administrador

Executar em todas as máquinas:

```sh
mkdir ~/.kube
sudo cat /etc/kubernetes/admin.conf | tee ~/.kube/config
```

## Configuração do _control plane_ como _worker_

Executar em qualquer máquina:

```sh
kubectl taint nodes --all node-role.kubernetes.io/control-plane:NoSchedule-
```
