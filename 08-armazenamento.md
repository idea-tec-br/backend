# 07: Armazenamento

Escolhido o [Longhorn](https://longhorn.io) para armazenamento.

## Instalação

Executar em todas as máquinas:

```sh
# Serviços básicos
sudo apt install -y open-iscsi nfs-common util-linux cryptsetup dmsetup
sudo systemctl enable --now iscsid
sudo modprobe iscsi_tcp
echo "iscsi_tcp" | sudo tee /etc/modules-load.d/iscsi.conf
```

Executar em qualquer máquina:

```sh
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/v1.12.0/deploy/longhorn.yaml
```
