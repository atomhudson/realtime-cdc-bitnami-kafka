# Grafana Dashboard Guide for CDC Pipeline Monitoring (MySQL → Kafka → ClickHouse)

## Overview

Grafana is an open-source analytics and interactive visualization web application. It's widely used to visualize time-series data for infrastructure and application analytics. In the context of a CDC (Change Data Capture) pipeline using Debezium, Kafka, MySQL, and ClickHouse, Grafana helps monitor:

- CDC throughput and lag
- Kafka topic activity
- MySQL binlog metrics
- ClickHouse ingestion rate and performance

---

## Components

- **Grafana Server**: Core service for dashboard rendering and data visualization.
- **Data Sources**: Integrations like Prometheus, MySQL, ClickHouse, Kafka exporters, etc.
- **Dashboards**: Panels to visualize metrics.
- **Plugins**: Extend Grafana's capability (e.g., ClickHouse plugin).
- **Alerts**: For threshold breaches in performance metrics.

---

## Prerequisites

Before setting up Grafana:

1. CDC setup using Debezium + Kafka + ClickHouse must be running.
2. Monitoring tools (recommended):
   - **Prometheus** (or other time-series DB) to scrape/export metrics.
   - Kafka Exporter (e.g., Bitnami’s JMX Exporter or Burrow).
   - ClickHouse Exporter (e.g., `Altinity/clickhouse_exporter`).
   - MySQL Exporter (e.g., `prom/mysqld-exporter`).

---

## Installation Guide

### Option 1: Docker (Recommended for Quick Start)

```bash
  docker run -d -p 3000:3000 --name=grafana grafana/grafana
```
### Access

- URL: `http://localhost:3000`
- Default credentials: `admin` / `admin`

---

## Connecting Data Sources

1. **Prometheus**:
   - URL: `http://<prometheus-ip>:9090`
   - Used for Kafka, MySQL, and ClickHouse metrics.

2. **ClickHouse**:
   - Install ClickHouse plugin from Grafana plugins directory.
   - Configure ClickHouse connection credentials.

3. **MySQL** (optional):
   - Native MySQL connector available.
   - Can be used to track schema/table changes directly.

---

## Useful Dashboards (References)

### Kafka Monitoring Dashboard
- Metrics:
  - Topic size
  - Consumer lag
  - Broker health
- Example: [Kafka Exporter Dashboard](https://grafana.com/grafana/dashboards/7589)

### Debezium Lag/Latency Dashboard
- Requires JMX Exporter on Kafka Connect
- Shows:
  - Source connector lag
  - Message throughput
  - Error count

### ClickHouse Performance Dashboard
- Use: Altinity ClickHouse Exporter
- Metrics:
  - Insert rates
  - Query time
  - Table sizes
- Example: [ClickHouse Dashboard](https://grafana.com/grafana/dashboards/12333)

---

## Use Cases

- **Monitor end-to-end CDC pipeline health**
- **Track latency from MySQL → Kafka → ClickHouse**
- **Detect failures or lags in CDC connectors**
- **Understand ClickHouse ingestion trends**
- **Display Kafka topic data volume and growth**
- **Set alerts on unusual activity or latency spikes**

---

## ✅ Summary

| Component     | Purpose                              |
|---------------|--------------------------------------|
| Grafana       | Visualization and alerts             |
| Prometheus    | Time-series metrics collection       |
| Kafka Exporter| Kafka topic/consumer metrics         |
| ClickHouse Exporter | ClickHouse insert/query stats |
| MySQL Exporter| Binlog and MySQL performance metrics |

---

## 📚 Resources

- [Grafana Docs](https://grafana.com/docs/)
- [Prometheus Docs](https://prometheus.io/docs/)
- [Debezium Monitoring](https://debezium.io/documentation/reference/stable/monitoring.html)
- [ClickHouse Exporter](https://github.com/Altinity/clickhouse_exporter)
- [Kafka Exporter](https://github.com/danielqsj/kafka_exporter)
