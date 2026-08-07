# 07: Armazenamento

Escolhido o [Longhorn](https://longhorn.io) para armazenamento.

## Instalação

Executar em todas as máquinas:

```sh
# Pacotes
sudo apt install -y open-iscsi nfs-common util-linux cryptsetup dmsetup
#
# Serviços
sudo systemctl disable --now multipathd.socket
sudo systemctl disable --now multipathd.service
sudo systemctl mask --now multipathd.service
sudo systemctl enable --now iscsid
#
# Módulos
cat <<EOF | sudo tee /etc/modules-load.d/iscsi.conf
nfs
dm_crypt
iscsi_tcp
EOF
sudo modprobe nfs
sudo modprobe dm_crypt
sudo modprobe iscsi_tcp
#
# Serviços: atualização
sudo systemctl restart iscsid
#
# Verificação do estado da máquina
longhornctl --kubeconfig=$HOME/.kube/config check preflight
```

Executar em qualquer máquina:

```sh
# Instalação
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/v1.12.0/deploy/longhorn.yaml
#
# Visualização local da interface Web
kubectl port-forward -n longhorn-system svc/longhorn-frontend 8080:80 
```
