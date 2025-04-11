# Debezium Installation Guide

## What is Debezium?
Debezium is an open-source distributed **Change Data Capture (CDC)** platform that allows applications to track changes in databases in real time. It captures changes from database logs and streams them into Kafka topics, making them available for consumers like ClickHouse, Elasticsearch, or other applications.

**Reference:** [Debezium Official Documentation](https://debezium.io/documentation/reference/3.1/)

## Why Use Debezium?
Debezium enables real-time data synchronization and event-driven architectures with:
- **Low Latency**: Captures and streams database changes instantly.
- **Reliability**: Ensures consistency by reading transaction logs instead of querying the database.
- **Scalability**: Works efficiently with distributed architectures.
- **Flexibility**: Supports various databases like MySQL, PostgreSQL, SQL Server, and MongoDB.

**Reference:** [Introduction to Debezium](https://debezium.io/documentation/reference/3.1/introduction.html)

## Installation Methods
Debezium can be installed in different ways, depending on the setup requirements:

### 1. Using Docker (Recommended)
This is the easiest and most portable way to run Debezium. It requires **Kafka** and **Kafka Connect**.

### 2. Running as a Standalone Service
Debezium can be run as an independent service with its own configuration and logging setup.

### 3. Using Kafka Connect (Embedded)
Debezium runs as a Kafka Connect source connector, making it easy to integrate with Kafka-based architectures.

## Setting Up Debezium with Docker Compose
Create a `docker-compose-debezium.yml` file to set up Debezium with Kafka, Zookeeper, and Kafka Connect.

```yml
version: '3.8'
services:
   connect:
      image: quay.io/debezium/connect:1.9
      container_name: debezium
      ports:
         - 8083:8083
      depends_on:
         - kafka
      links:
         - kafka
         - zookeeper
      environment:
         - BOOTSTRAP_SERVERS=kafka:9092
         - GROUP_ID=1
         - CONFIG_STORAGE_TOPIC=my_connect_configs
         - OFFSET_STORAGE_TOPIC=my_connect_offsets
         - STATUS_STORAGE_TOPIC=my_connect_statuses
   debezium-ui:
      image: debezium/debezium-ui:latest
      container_name: debezium-ui
      ports:
         - 8084:8080
      depends_on:
         - connect
      environment:
         - KAFKA_CONNECT_URIS=http://connect:8083

```

Start the services:
```sh
docker-compose -f docker-compose-debezium.yml up
```

**Reference:** [Debezium with Docker](https://debezium.io/documentation/reference/3.1/operations/installation.html)

## Configuring the Debezium Connector for MySQL
Once Debezium is running, register the MySQL connector using the following JSON configuration (`mysql-connector.json`):

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

### Register the connector:
```sh
POST http://localhost:8083/connectors/
```
### Check status of connector
**Endpoint**
```bash
GET http://localhost:8083/connectors/mysql-connector/status
```
**Expected output:**
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
### Restart Sink Connector

```bash
POST http://localhost:8083/connectors/mysql-connector/restart
```
### Delete the Sink Connector
```bash 
DELETE http://localhost:8083/connectors/mysql-connector
```

**Reference:** [Debezium MySQL Connector](https://debezium.io/documentation/reference/3.1/connectors/mysql.html)

## Verifying Debezium is Capturing Changes
To ensure Debezium is capturing changes:
1. **Check Registered Connectors**
   ```sh
   curl -s http://localhost:8083/connectors 
   ```
2. **Consume Kafka Messages**
   ```sh
   docker run --rm --net=host confluentinc/cp-kafkacat kafkacat -b localhost:9092 -t inventory -C -o beginning
   ```

**Reference:** [Debezium Monitoring](https://debezium.io/documentation/reference/3.1/operations/monitoring.html)

**Further Reading:**
- [Debezium Configuration](https://debezium.io/documentation/reference/3.1/configuration.html)
- [Kafka Connect Configuration](https://docs.confluent.io/platform/current/connect/index.html)
- [MySQL Binlog Setup](https://dev.mysql.com/doc/refman/8.0/en/binary-log.html)

