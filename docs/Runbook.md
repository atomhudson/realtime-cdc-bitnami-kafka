# CDC Pipeline Runbook

## Prerequisites

- Docker & Docker Compose installed
- External Docker network created:
  ```bash
  docker network create cdc-network
  ```


## Overview

This document explains how to:
- Start all CDC pipeline services
- Configure the **Debezium MySQL Source Connector**
- Configure the **ClickHouse Sink Connector**

---

## Docker Compose Files

Your CDC pipeline uses four Docker Compose files:

| File | Purpose |
|------|---------|
| `docker-compose-mysq.yml` | MySQL CDC Source |
| `docker-compose-kafka.yml` | Kafka, Zookeeper, Kafka UI |
| `docker-compose-debezium.yml` | Debezium Connect + Debezium UI |
| `docker-compose-clickhouse.yml` | ClickHouse Sink |


---

## Starting the Services

**Run in this order** to respect dependencies.

1. **MySQL**
   ```bash
   docker-compose -f docker-compose-mysq.yml up -d
   ```

2. **Kafka & Zookeeper**
   ```bash
   docker-compose -f docker-compose-kafka.yml up -d
   ```

3. **Debezium Connect & UI**
   ```bash
   docker-compose -f docker-compose-debezium.yml up -d
   ```

4. **ClickHouse**
   ```bash
   docker-compose -f docker-compose-clickhouse.yml up -d
   ```
### Alternative: Start All Services Together

You can also start all services at once using:

```bash
docker-compose -f docker-compose-mysq.yml -f docker-compose-kafka.yml -f docker-compose-debezium.yml -f docker-compose-clickhouse.yml up -d
```

---

## Configure Debezium MySQL Source Connector & Clickhouse Sink Connector

Use Postman or CURL to register the source connector:

### To check all the available connectors
**Endpoint:**
```
GET http://localhost:8083/connectors
```
### To Configure the connectors
**Endpoint:**
```
POST http://localhost:8083/connectors
```

**Payload:**
### Source Connector: (MySQL)
```json
{
  "name": "mysql-connector",
  "config": {
    "connector.class": "io.debezium.connector.mysql.MySqlConnector",
    "tasks.max": "1",
    "database.hostname": "host.docker.internal",
    "database.port": "3307",
    "database.user": "debezium",
    "database.password": "password",
    "database.server.id": "223344",
    "database.server.name": "mysql_testdb_server",
    "database.include.list": "testdb",
    "table.include.list": "testdb.user,testdb.demo",
    "database.history.kafka.bootstrap.servers": "kafka:9092",
    "database.history.kafka.topic": "mysql_testdb_server.schema-changes",
    "database.connection.url": "jdbc:mysql://host.docker.internal:3307/testdb?allowPublicKeyRetrieval=true&useSSL=false&serverTimezone=UTC",
    "database.history.skip.unreachable.databases": "true",
    "snapshot.locking.mode": "none"
  }
}
```

> Replace `testdb` with the table you want to capture changes from.

> We use `host.docker.internal` because of our mysql service is running on a docker compose on port `3307`. So change this accordingly.

---
### Sink Connector: (Clickhouse)
```json
{
  "name": "clickhouse-sink-connector", 
  "config": {
    "connector.class": "com.clickhouse.kafka.connect.ClickHouseSinkConnector",
    "tasks.max": "1",

    "topics": "mysql_testdb_server.testdb.user,mysql_testdb_server.testdb.demo",

    "hostname": "clickhouse",
    "port": "8123",
    "database": "cdc_sink",
    "username": "default",
    "password": "",
    "batch.size": "20000",
    "linger.ms": "200",

    "value.converter": "org.apache.kafka.connect.json.JsonConverter",
    "value.converter.schemas.enable": "true",

    "transforms": "unwrap",
    "transforms.unwrap.type": "io.debezium.transforms.ExtractNewRecordState",
    "transforms.unwrap.drop.tombstones": "true",
    "transforms.unwrap.delete.handling.mode": "drop",


    "errors.tolerance": "all",
    "errors.log.enable": "true"
  }
}

```
###  To check connector is running or not
**Endpoint:**
```
GET http://localhost:8083/connectors/mysql-connector/status
```
```json
{
    "name": "mysql-connector",
    "connector": {
        "state": "RUNNING",
        "worker_id": "172.19.0.7:8083"
    },
    "tasks": [
        {
            "id": 0,
            "state": "RUNNING",
            "worker_id": "172.19.0.7:8083"
        }
    ],
    "type": "source"
}
```

**Endpoint:**
```
GET http://localhost:8083/connectors/clickhouse-sink-connector/status
```
**Expected output:**
```json
{
    "name": "clickhouse-sink-connector",
    "connector": {
        "state": "RUNNING",
        "worker_id": "172.19.0.7:8083"
    },
    "tasks": [
        {
            "id": 0,
            "state": "RUNNING",
            "worker_id": "172.19.0.7:8083"
        }
    ],
    "type": "sink"
}
```
### To delete or remove a connector 
**Endpoint:**
```
DELETE http://localhost:8083/connectors/<connector-name>
```

### To restart a connector

**Endpoint:**
```
POST http://localhost:8083/connectors/<connector-name>/restart
```
> Note : **<connector-name>** change accordingly like in our case **"mysql-connector","clickhouse-sink-connector"**.

---

## 📊 Kafka UI Dashboard

Access Kafka UI at:
```
http://localhost:8080
```

Use it to:
- Monitor topics (e.g. `mysqlserver1.testdb.your_table_name`)
- Validate Debezium publishing

---

## ✅ Validation Checklist

- [ ] MySQL container is up and logging binary logs.
- [ ] Kafka is running and accessible on port `9092`.
- [ ] Debezium Connect is running on port `8083` and plugin path is correctly mounted.
- [ ] Debezium UI is accessible on port `8084`.
- [ ] ClickHouse is up and port `8123` is available.
- [ ] Topics are created in Kafka and data is flowing from MySQL → Kafka → ClickHouse.

---