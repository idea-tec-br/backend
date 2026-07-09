# 06: _Container Network Interface_ (CNI)

Escolhido o [Cilium](https://cilium.io/) como [_add-on_ de rede](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#pod-network).

## Instalação

```sh
# Instalação: Cilium
cilium install -f cilium.yaml
kubectl delete daemonset kube-proxy -n kube-system
#
# Ativação: Hubble
cilium hubble enable
#
# Monitoramento: Cilium
cilium status --wait
#
# Monitoramento: Hubble
hubble status -P
hubble observe -P
#
# Teste: Cilium + Hubble
cilium connectivity test
```
