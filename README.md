### **ClickHouse Basics**
- What is ClickHouse?
- How does ClickHouse work?
- Why is ClickHouse useful in real-time data sync?
- Key use cases of ClickHouse
- Difference between OLTP (e.g., MySQL) and OLAP (e.g., ClickHouse)

---

### **MergeTree & Engine Types in ClickHouse**
- What is `MergeTree` in ClickHouse?
- Key features of MergeTree:
  - Primary key indexing
  - Partitioning
  - Deduplication, TTL, Sampling
  - Background part merging
- How MergeTree organizes and merges data internally (requested diagram)

---

### **MergeTree Variants / Table Engines**
- What each engine does and when to use them
- Visual example of how each behaves on sample data (requested diagram)

---

### **OLTP vs OLAP**
- Full comparison: OLTP (MySQL, PostgreSQL) vs OLAP (ClickHouse, Druid)
- Use cases for both and how they complement each other
- Real-world scenario combining OLTP & OLAP (requested diagram)

---

### **Real-Time Data Pipeline**
- How ClickHouse fits into a real-time data sync pipeline
- Architecture: MySQL → Debezium → Kafka → ClickHouse

---
- What is Debezium
- Difference Between Debezium and Debezium CDC
- Why It Is Useful in OLAP and Its Use Cases


Debezium Engine
Data catalogs
Debezium Config
Load Testing (JMETER)

