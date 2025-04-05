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


- Debezium Engine
- Data catalogs
- Debezium Config
- Load Testing (JMETER)

- ### Unit Testing
- This is the smallest scale of testing and focuses on individual units or components (smallest parts) of the software. During development, developers test these units.

### Integration Testing
- It involves testing the interaction between different units or modules to ensure they work together as intended when integrated.

### System Testing
- This type of testing verifies that the entire system or application meets the specified requirements. It tests the system as a whole and checks whether it properly meets functional or non-functional requirements.

### Regression Testing
- It ensures that recent code changes haven't affected existing functionalities.


