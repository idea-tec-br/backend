# 10: Testes

Testes de integração entre os serviços.

## HTTP/1

```sh
kubectl apply -f teste-0.yaml
curl -4 -v http://teste-infra.k8s.sj.ifsc.edu.br
curl -6 -v http://teste-infra.k8s.sj.ifsc.edu.br
kubectl delete -f teste-0.yaml
```

## HTTP/2

```sh
kubectl apply -f teste-1.yaml
curl -4 -v --http2 https://teste-infra.k8s.sj.ifsc.edu.br
curl -6 -v --http2 https://teste-infra.k8s.sj.ifsc.edu.br
kubectl delete -f teste-1.yaml
```

## Armazenamento

```sh
# Criação dos recursos
kubectl apply -f teste-2.yaml
#
# Dashboard
kubectl port-forward -n longhorn-system svc/longhorn-frontend 8080:80
#
# Modificação de arquivo
kubectl exec -n teste-infra longhorn-test-pod -- curl -s http://localhost
kubectl exec -n teste-infra longhorn-test-pod -- sh -c "echo 'Longhorn está vivo e replicando!' > /usr/share/nginx/html/index.html"
kubectl exec -n teste-infra longhorn-test-pod -- curl -s http://localhost
#
# Recriação de pod
kubectl delete -n teste-infra pod longhorn-test-pod
kubectl apply -f teste-2.yaml
#
# Verificação do conteúdo modificado
kubectl exec -n teste-infra longhorn-test-pod -- curl -s http://localhost
#
# Remoção dos recursos
kubectl delete -f teste-2.yaml
```
