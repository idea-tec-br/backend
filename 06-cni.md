# 06: _Container Network Interface_ (CNI)

Escolhido o [Cilium](https://cilium.io/) como [_add-on_ de rede](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#pod-network).

## Instalação

```sh
# Instalação
cilium install -f cilium.yaml
kubectl delete daemonset kube-proxy -n kube-system
#
# Monitoramento
cilium status --wait
#
# Teste
cilium connectivity test
cilium connectivity test --cleanup
```
