# Serviços

Este *cluster* possui, por enquanto, dois serviços: *broker* de mensagens (MQTT) e banco de dados não relacional baseado em séries temporais (TSDB).

O arquivo `0-common.yaml` possui recursos destinados a todos os serviços: 

- *namespace*: `Namespace`;
- Certificação digital e *proxy*: `ClusterIssuer` e `Gateway`.

## MQTT: Mosquitto

O arquivo `mqtt.yaml` tem todos os recursos necessários.

## TSDB: InfluxDB

Além do arquivo `tsdb.yaml`, O InfluxDB requer um `Secret` para operar, no seguinte formato:

```yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: secret-influxdb
  namespace: feira-de-jogos
type: Opaque
data:
  influxdb-username: ZmVpcmE= # echo -n 'feira' | base64
  influxdb-password: ZmVpcmE= # echo -n 'feira' | base64
  influxdb-admin-token: ZmVpcmE= # echo -n 'feira' | base64
```

onde `influxdb-password` e `influxdb-admin-token` devem ser atualizados.