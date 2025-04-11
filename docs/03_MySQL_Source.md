# MySQL as a Source for Debezium CDC

## Overview

The **Debezium MySQL Connector** is a key component in Change Data Capture (CDC) pipelines, enabling real-time streaming of MySQL database changes to Apache Kafka. It reads MySQL's binary logs (binlogs) to capture inserts, updates, and deletes, making it essential for event-driven architectures and real-time analytics.

## How MySQL Acts as a Source in CDC Pipelines

MySQL acts as a source database for Debezium by leveraging its binary logging mechanism to track all changes made to tables. The Debezium MySQL Connector captures these changes and streams them into Kafka topics. This enables real-time data synchronization, event-driven workflows, and analytics use cases.

Debezium integrates with MySQL as a source by:

- Connecting to the MySQL server and monitoring its binlogs.
- Capturing row-level changes in real-time and streaming them to Kafka topics.
- Providing an immutable stream of change events, preserving the order of transactions.

## Installation Process (Docker-Based)

### 1. Docker Compose Method:
 `docker-compose` to manage MySQL:  

1. **Create a `docker-compose-mysql.yml` file:**  

```yaml
version: '3.8'
services:
  mysql:
    image: mysql:8.0
    container_name: mysql-cdc
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: testdb
    ports:
      - "3307:3306"
    volumes:
      - mysql_data:/var/lib/mysql
    command: >
      --server-id=1
      --log-bin=mysql-bin
      --binlog-format=ROW
      --binlog-row-image=FULL
      --enforce-gtid-consistency=ON
      --gtid-mode=ON

volumes:
  mysql_data:

```

2. **Run the following command to start MySQL:**  
```sh
docker-compose up -d
```

This method makes it easier to manage, modify, and scale the setup. Let me know if you need any refinements! 🚀

### 2. Create a Debezium User in MySQL

Connect to the MySQL instance:

```sh
docker exec -it mysql-cdc mysql -uroot -proot
```

Then run the following SQL commands to create a user for Debezium with the necessary privileges:

```sql
CREATE USER 'debezium'@'%' IDENTIFIED BY 'dbz';
GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'debezium'@'%';
FLUSH PRIVILEGES;
```

Exit MySQL:
```sh
exit
```

## Performance Considerations

- **Disk Space Management**: Large binlogs can impact storage. Tune retention settings appropriately.
- **Replication & Failover**: If using MySQL replication, ensure Debezium reads from a replica to reduce load on the primary.
- **Kafka Partitioning**: Ensure proper partitioning strategy to handle large data streams efficiently.

## Connector Configuration

### Essential Configuration Options
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

## Transaction Logs in MySQL

### MySQL Binlogs

- MySQL uses **binary logs (binlogs)** to record all changes to database tables.
- Binlogs capture events in a sequential manner, enabling replication and CDC.
- Debezium reads binlogs and converts changes into Kafka events.

## Checking and Enabling Binlogs in MySQL

### Checking Binlog Status

To check if binlogs are enabled, run:

```sh
docker exec -it mysql-cdc mysql -uroot -proot -e "SHOW VARIABLES LIKE 'log_bin';"
```

If `log_bin` is set to `ON`, binlogs are enabled.

### Enabling Binlogs

If binlogs are not enabled, restart the MySQL container with the correct configuration as mentioned in the installation section.

## MySQL Binary Log Configuration

---

### Verifying Binary Log (Binlog) Status

```sql
SHOW VARIABLES LIKE 'log_bin';
SHOW VARIABLES LIKE 'binlog_format';
SHOW VARIABLES LIKE 'binlog_row_image';
```

### Verifying Current Binlog Position (for the current session)

```sql
SHOW MASTER STATUS;
```

### Viewing Available Binary Logs

```sql
SHOW BINARY LOGS;
```

### Inspecting a Specific Binary Log

Replace `'mysql-bin.000xxx'` with the actual log file name:

```sql
SHOW BINLOG EVENTS IN 'mysql-bin.000xxx';
```
---

### Connect to MySQL Service

```bash
mysql -h 127.0.0.1 -P 3307 -u root -p
```

> **Note:** When prompted, enter the password: `root`.  
> If the password is incorrect, either reset the credentials or create a new user and grant the required permissions.

---

### Create Tables

> Our MySQL service (running on port `3307`) automatically creates a database named `testdb`.

After connecting to the MySQL service:

#### Check Available Databases

```sql
SHOW DATABASES;
```

#### Use the Database and Create Tables

```sql
USE testdb;

CREATE TABLE user (
  id INT PRIMARY KEY,
  name VARCHAR(100)
);

CREATE TABLE demo (
  id INT PRIMARY KEY,
  address VARCHAR(100)
);
```

---
## Next Steps

After configuring MySQL as a source, proceed with setting up Kafka and Debezium Connector to start streaming data. 
For further details, refer to the [Debezium MySQL Connector Documentation](https://debezium.io/documentation/reference/3.1/connectors/mysql.html#debezium-connector-for-mysql).

