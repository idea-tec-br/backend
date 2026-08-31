# Serviços

Este *cluster* possui, por enquanto, dois serviços: *broker* de mensagens (MQTT) e banco de dados não relacional baseado em séries temporais (TSDB).

O arquivo `0-common.yaml` possui recursos destinados a todos os serviços: 

- *namespace*: `Namespace`;
- Certificação digital e *proxy*: `ClusterIssuer` e `Gateway`.

## MQTT: Mosquitto

O arquivo `mqtt.yaml` tem todos os recursos necessários.

## TSDB: InfluxDB

Além do arquivo `tsdb.yaml`, O InfluxDB requer um `Secret` para operar.

## REST API

Além do arquivo `rest-api.yaml`, o REST API requer um `Secret` para operar.

## Secrets

```yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: secret-influxdb
  namespace: feira-de-jogos
type: Opaque
stringData:
  influxdb-admin-token: <token>
  influxdb-password: <password>
  influxdb-username: <username>
---
apiVersion: v1
kind: Secret
metadata:
  name: secret-rest-api
  namespace: feira-de-jogos
type: Opaque
stringData:
  PGHOST: <hostname>
  PGPORT: <port>
  PGDATABASE: <database>
  PGUSER: <username>
  PGPASSWORD: <passqord>
  PORT: <port>
  GOOGLE_CLIENT_ID: <list>
  GOOGLE_CLIENT_ID_GAMES: <list>
  GOOGLE_CLIENT_ID_GAMES_20241: <list>
  TOKEN_SECRET_KEY_ARCADE: <key>
  TOKEN_SECRET_KEY_VENDING_MACHINE: <key>
  MQTT_BROKER_URL: <url>
  MQTT_CLIENT_ID: <id>
  MQTT_TOPIC_STATUS: <topic>
  MQTT_TOPIC_COMANDO_ID_1: <topic>
  MQTT_TOPIC_COMANDO_ID_2: <topic>
  MQTT_TOPIC_ESTOQUE_SET_ID_1: <topic>
  MQTT_TOPIC_ESTOQUE_SET_ID_2: <topic>
  MQTT_TOPIC_ESTOQUE_REQUEST: <topic>
  MQTT_MACHINE_ID_1: <id>
  MQTT_MACHINE_ID_2: <id>
```
