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
  DOCKER_INFLUXDB_INIT_MODE: <valor>
  DOCKER_INFLUXDB_INIT_ORG: <valor>
  DOCKER_INFLUXDB_INIT_BUCKET: <valor>
  DOCKER_INFLUXDB_INIT_USERNAME: <valor>
  DOCKER_INFLUXDB_INIT_PASSWORD: <valor>
  DOCKER_INFLUXDB_INIT_ADMIN_TOKEN: <valor>
---
apiVersion: v1
kind: Secret
metadata:
  name: secret-rest-api
  namespace: feira-de-jogos
type: Opaque
stringData:
  PGHOST: <valor>
  PGPORT: <valor>
  PGDATABASE: <valor>
  PGUSER: <valor>
  PGPASSWORD: <valor>
  PORT: <valor>
  GOOGLE_CLIENT_ID: <valor>
  GOOGLE_CLIENT_ID_GAMES: <valor>
  GOOGLE_CLIENT_ID_GAMES_20241: <valor>
  TOKEN_SECRET_KEY_ARCADE: <valor>
  TOKEN_SECRET_KEY_VENDING_MACHINE: <valor>
  MQTT_BROKER_URL: <valor>
  MQTT_CLIENT_ID: <valor>
  MQTT_TOPIC_STATUS: <valor>
  MQTT_TOPIC_COMANDO_ID_1: <valor>
  MQTT_TOPIC_COMANDO_ID_2: <valor>
  MQTT_TOPIC_ESTOQUE_SET_ID_1: <valor>
  MQTT_TOPIC_ESTOQUE_SET_ID_2: <valor>
  MQTT_TOPIC_ESTOQUE_REQUEST: <valor>
  MQTT_MACHINE_ID_1: <valor>
  MQTT_MACHINE_ID_2: <valor>
---
apiVersion: v1
kind: Secret
metadata:
  name: secret-db
  namespace: feira-de-jogos
stringData:
  POSTGRES_DB: <valor>
  POSTGRES_USER: <valor>
  POSTGRES_PASSWORD: <valor>
```
