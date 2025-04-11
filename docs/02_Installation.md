# Real-Time Data Synchronization - Installation Guide

## Overview
This guide provides step-by-step instructions for setting up a real-time data synchronization system using Debezium CDC, MySQL as the source database, Kafka as the streaming platform, and ClickHouse as the sink. The installation process is divided into multiple sections for better organization and ease of implementation.

By following this guide, you will be able to:
- Set up MySQL as the source database with Change Data Capture (CDC) enabled.
- Install and configure Kafka for real-time event streaming.
- Deploy Debezium as a Kafka Connect source to capture changes from MySQL.
- Configure ClickHouse as a sink to store and process streaming data efficiently.

Sure! Here's your original **Prerequisites** section converted to the new concise format you mentioned:

---

### **Prerequisites**

- **Docker**: Ensure Docker and Docker Compose are installed and operational.  
- **MySQL**: Use MySQL 8.0+ with binary logging enabled and accessible to Debezium.  
- **Kafka**: Ensure Kafka and Kafka Connect are up and running.  
- **ClickHouse**: Confirm ClickHouse is installed and reachable.  
- **Debezium Connector**: Add the Debezium MySQL Source Connector to Kafka Connect.  
  ```bash
  curl -O https://repo1.maven.org/maven2/io/debezium/debezium-connector-mysql/2.6.1.Final/debezium-connector-mysql-2.6.1.Final-plugin.tar.gz
  tar -xzf debezium-connector-mysql-2.6.1.Final-plugin.tar.gz -C <PLUGIN_PATH>
  ```
- **MySQL Permissions**: Ensure the MySQL user has privileges to read binlogs:
  ```sql
  GRANT SELECT, RELOAD, SHOW DATABASES, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'debezium'@'%';
  ```

## Installation Steps
To set up the complete system, follow these individual installation guides:

1. **MySQL as Source**  
   - Setup MySQL database and configure it for Debezium CDC.
   - Enable binlog and configure necessary settings.
   - [Read MySQL Source Installation Guide](https://github.com/Datavolt/debezium-cdc/blob/main/docs/03_MySQL_Source.md)

2. **Kafka Installation**  
   - Install and configure Kafka for event streaming.
   - Setup Zookeeper and Kafka brokers.
   - [Read Kafka Installation Guide](https://github.com/Datavolt/debezium-cdc/blob/main/docs/04_kafka_installation.md)

3. **Debezium Installation**  
   - Install Debezium and configure it as a Kafka Connect source.
   - Setup Debezium connectors for MySQL.
   - [Read Debezium Installation Guide](https://github.com/Datavolt/debezium-cdc/blob/main/docs/06_debezium_installation.md)

4. **ClickHouse as Sink**  
   - Install ClickHouse and configure it to consume Kafka topics.
   - Setup ClickHouse Kafka engine for data ingestion.
   - [Read ClickHouse Sink Installation Guide](https://github.com/Datavolt/debezium-cdc/blob/main/docs/05_Clickhouse_Sink.md)

---
   
## Conclusion
Once you have completed the installation and setup of MySQL, Kafka, Debezium, and ClickHouse, your real-time data synchronization system will be ready. 

To ensure smooth operation:
- Monitor Kafka topics and ensure they are correctly streaming data.
- Regularly check MySQL binlogs and Debezium connector status.
- Optimize ClickHouse ingestion performance for high throughput.

For further customization and advanced configurations, refer to the detailed installation guides linked above.



