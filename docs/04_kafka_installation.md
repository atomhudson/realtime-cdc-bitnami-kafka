# Kafka Installation Guide

## Installation Methods

### 1. Using Docker Compose (Recommended)
The easiest way to set up Bitnami Kafka is by using Docker Compose. Below is a **Docker Compose configuration** that sets up Kafka with ZooKeeper using **Bitnami images**:

#### Step 1: Create a `docker-compose-kafka.yml` file

```yml
version: '3.8'

services:
  zookeeper:
    image: bitnami/zookeeper:latest
    container_name: zookeeper
    restart: unless-stopped
    ports:
      - "2181:2181"
    environment:
      ALLOW_ANONYMOUS_LOGIN: yes
    networks:
      - cdc-network

  kafka:
    image: bitnami/kafka:3.5.1
    container_name: kafka
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
      - "7071:7071" # JMX Exporter port
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_CFG_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_CFG_LISTENERS: PLAINTEXT://:9092
      KAFKA_CFG_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092
      KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT
      KAFKA_CFG_AUTO_CREATE_TOPICS_ENABLE: "true"
      KAFKA_CFG_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_LOG4J_LOGGERS: "kafka.controller=INFO,kafka.producer.async.DefaultEventHandler=INFO,state.change.logger=INFO"
      ALLOW_PLAINTEXT_LISTENER: yes
      KAFKA_JMX_PORT: 9999
      JMX_PORT: 9999
      KAFKA_OPTS: "-javaagent:/opt/jmx_exporter/jmx_prometheus_javaagent.jar=7071:/opt/jmx_exporter/kafka.yml"
    volumes:
      - ./<path-to-jmx-exporter-folder>/jmx_prometheus_javaagent-0.19.0.jar:/opt/jmx_exporter/jmx_prometheus_javaagent.jar
      - ./<path-to-jmx-exporter-folder>/kafka-2_0_0.yml:/opt/jmx_exporter/kafka.yml
    networks:
      - cdc-network

  kafka-ui:
    image: provectuslabs/kafka-ui:latest
    container_name: kafka-ui
    restart: unless-stopped
    depends_on:
      - kafka
    ports:
      - "8080:8080"
    environment:
      KAFKA_CLUSTERS_0_NAME: local-kafka
      KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: kafka:9092
      KAFKA_CLUSTERS_0_ZOOKEEPER: zookeeper:2181
    networks:
      - cdc-network

networks:
  cdc-network:
    external: true
```
> **Note:** Replace `<path-to-jmx-exporter-folder>` with the actual path where you've placed the `kafka-2_0_0.yml` and `jmx_prometheus_javaagent-0.19.0.jar` files.
 

#### Step 2: Start Kafka
Run the following command in the same directory where `docker-compose.yml` is located:

```bash
docker-compose -f docker-compose-kafka.yml up
```

#### Step 3: Verify Kafka is Running
To check the logs and confirm that Kafka has started successfully:

```sh
docker logs -f kafka
```

**Reference:** [Bitnami Kafka Docker Hub](https://hub.docker.com/r/bitnami/kafka), [Bitnami Zookeeper Docker Hub](https://hub.docker.com/r/bitnami/zookeeper)

---

## Understanding Kafka

### What is Kafka?
Kafka is a **distributed event streaming platform** designed for high-throughput, fault-tolerant data pipelines, event-driven architectures, and real-time analytics.

### Why Use Kafka?
- **Scalability:** Handles high volumes of data efficiently.
- **Fault Tolerance:** Replicates data across multiple nodes.
- **Real-Time Streaming:** Processes data in real-time for instant insights.
- **Decoupled Architecture:** Allows services to communicate asynchronously without tight coupling.
- **Guaranteed Message Delivery:** Provides durability through log-based storage and replication.

**Reference:** [Kafka Documentation](https://kafka.apache.org/documentation/)

### Kafka Topics
Kafka stores messages in **topics**, which are:
- **Partitioned** for scalability.
- **Replicated** for fault tolerance.

**Partitions** allow Kafka to parallelize message processing across multiple consumers.

### Kafka Producers & Consumers
- **Producers** send messages to Kafka topics asynchronously.
- **Consumers** read messages from Kafka topics at their own pace, supporting both real-time and batch processing.

### What is ZooKeeper & Its Role in Kafka?
ZooKeeper helps Kafka manage:
- **Leader election** for partitioned topics.
- **Configuration management** for brokers.
- **Health monitoring** of Kafka nodes.

**Kafka Without ZooKeeper (KRaft Mode):**
- Kafka 3.x introduces KRaft (Kafka Raft) mode, removing the need for ZooKeeper and improving scalability.

**Reference:** [ZooKeeper Overview](https://zookeeper.apache.org/doc/current/zookeeperOver.html)

### Why Kafka is Essential for This Pipeline
- **Scalability:** Kafka efficiently handles large-scale data changes.
- **Fault Tolerance:** Messages remain in Kafka even if ClickHouse is temporarily unavailable.
- **Event Replay:** Kafka allows reprocessing past CDC events if needed.
- **Decoupled Processing:** MySQL changes can be processed independently by multiple services (analytics, logging, monitoring, etc.).
- **High Throughput:** Kafka ensures high-speed data ingestion and delivery.

**Reference:** [Debezium Kafka Integration](https://debezium.io/documentation/reference/3.1/tutorial.html)

---

### Additional Considerations
- **Security:** Set up authentication & authorization via SSL, SASL, and ACLs.
- **Monitoring:** Use tools like Prometheus, Grafana, or Confluent Control Center.
- **Retention Policies:** Configure log retention for optimized storage.
- **Replication Factors:** Ensure high availability by setting replication factors appropriately.

---

## Next Steps
- Set up **Kafka Connect** to integrate with external systems.
- Configure **Debezium** to use Kafka for real-time data streaming.
- Tune Kafka performance for production use.

**Further Reading:**
- [Kafka Streams](https://kafka.apache.org/documentation/streams/)
- [Kafka Connect](https://docs.confluent.io/platform/current/connect/index.html)
- [Kafka Security](https://docs.confluent.io/platform/current/kafka/security-overview.html)
