# 08: COntrole remotoApenas na primeira máquina:

Escolhidos [k9s](https://k9scli.io/) para CLI e [Headlamp](https://headlamp.dev/) para Web GUI.

## Instalação

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
