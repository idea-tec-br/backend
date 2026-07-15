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
