# 02: _Container Runtime Interface_ (CRI)

Escolhido o [containerd](https://containerd.io/) como [CRI](https://kubernetes.io/docs/setup/production-environment/container-runtimes/).

## Instalação

Executar em todas as máquinas:

```sh
sudo apt install -y containerd
sudo install -d /etc/containerd
containerd config default | sed 's/SystemdCgroup = false/SystemdCgroup = true/g' |sudo tee /etc/containerd/config.toml > /dev/null
sudo systemctl restart containerd
```
