# 05: _Gateway API_

Escolhido o [Gateway API](https://kubernetes.io/docs/concepts/services-networking/gateway/) para tráfego externo.

## Instalação

Executar em qualquer máquina:

```sh
kubectl apply --server-side -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.6.0/experimental-install.yaml
```