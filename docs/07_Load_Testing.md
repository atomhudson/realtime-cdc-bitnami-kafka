## Testing

### Performance Testing
- This type of testing evaluates the system's performance under various conditions, such as load, stress, and scalability. It helps identify bottlenecks and assesses the system's responsiveness and stability.

### Load Testing
- It is a subset of performance testing. It involves testing the system's behavior under specific expected load conditions. A popular tool for this is **JMeter**.
---
# Load Testing

- Load testing involves checking how a system behaves under expected user load.
- The goal is to measure system performance such as response time, stability, and resource usage.
- A popular tool for load testing is **Apache JMeter**.

## Why JMeter?
- JMeter allows you to simulate multiple users (called *threads*) sending requests to a server.
- It helps evaluate how the server performs under different load conditions.
## What is a Test Plan in JMeter?
- A Test Plan is like a recipe that tells JMeter what to test and how to test it.
- It includes everything needed to simulate how users interact with your application — such as logging in, clicking buttons, or making API calls.

---
## Key Features of JMeter

### Protocol Support
- JMeter supports many protocols, making it suitable for various types of applications:
  - **HTTP, HTTPS**
  - **JDBC** (for database testing)
  - **FTP**
  - **SOAP & REST APIs**
  - and more.

### Test Plans
- Tests are structured using **Test Plans**.
- A test plan can include:
  - **Thread Groups** – define the number of users and their behavior.
  - **Samplers** – simulate requests (e.g., HTTP request).
  - **Listeners** – collect and display test results.
  - **Assertions & Config Elements** – to check conditions and configure requests.

### High Configurability
- JMeter lets you customize:
  - Request data
  - Parameters
  - Timing (delays, ramp-up time)
  - Assertions (conditions to validate)
- This helps simulate real-world scenarios accurately.

### User-Friendly Interface
- Provides a **GUI (Graphical User Interface)** for creating and running test plans easily.
- Also supports:
  - **Command-line mode** for headless execution
  - **Distributed testing** for simulating heavy load using multiple machines

### Reporting & Analysis
- Includes multiple listeners like:
  - Graph Results
  - Summary Report
  - Aggregate Report
  - View Results Tree
- Helps you identify:
  - Performance bottlenecks
  - Failures and slow requests
  - Trends under different loads

### Extensibility
- Supports a wide range of **plugins**.
- Can be extended with:
  - Custom scripts
  - Community and third-party plugins
- Useful for advanced or specialized testing needs.

---
## Prerequisites
Before installing and using JMeter for load testing, make sure the following prerequisites are met:
---

### Java (Required for JMeter)

- JMeter is a Java-based application, so **Java Development Kit (JDK)** must be installed on your system.

- [Download Java (JDK)](https://www.oracle.com/java/technologies/downloads/)

- After installation, verify that Java is installed correctly by running:

  ```bash
  java -version
  ```
### Docker (For CDC Stack)

If you're running the **CDC system (Debezium, Kafka, MySQL, ClickHouse)** using Docker Compose:

- Ensure the following ports are exposed correctly:
  - **MySQL**: `3306`
  - **Kafka**: `9092`
  - **ClickHouse**: `8123`

- Enable **Docker health checks** to ensure containers (especially Kafka and MySQL) are fully ready before starting load tests.

- Monitor resource usage during tests using:

  ```bash
  docker stats
  ```
---

## Steps to Install JMeter

### 1. Download JMeter

- Visit the official JMeter website: [https://jmeter.apache.org/](https://jmeter.apache.org/)
- Go to the **Download** section.
- Download the **latest binary version** of JMeter.  
  [Direct download link](https://jmeter.apache.org/download_jmeter.cgi)

---

### 2. Extract JMeter Files

- Once the ZIP file is downloaded, extract it to a location of your choice.  
  Example path:  
  `C:\apache-jmeter-X.XX` (on Windows)  
  or  
  `/home/user/apache-jmeter-X.XX` (on Linux/Mac)

---

### 3. Run JMeter

- Navigate to the extracted JMeter folder.
- Open the `bin` directory.

You’ll see several files including:

- `jmeter.bat` – for **Windows GUI mode**
- `jmeter` – for **Linux/Mac GUI mode**
- `jmeter -n` – for **non-GUI (command-line) mode**

---

### 4. Start JMeter

#### For GUI Mode:
- On Windows: Double-click `jmeter.bat`
- On Mac/Linux: Run `./jmeter` in terminal

This opens the JMeter **Graphical User Interface**, where you can build and execute test plans.

#### For Non-GUI Mode (Command-line Testing):
- Open a terminal or command prompt.
- Run a test plan like this:
  ```bash
  jmeter -n -t your_test_plan.jmx -l results.jtl
  ```
  - `-n` → Non-GUI mode  
  - `-t` → Path to your `.jmx` test plan  
  - `-l` → Log file for results
---

# Load Testing a CDC System with JMeter

This guide explains how to **perform load testing using Apache JMeter** on a **Change Data Capture (CDC) pipeline** built with:

- **MySQL** as source
- **Debezium** for capturing changes
- **Bitnami Kafka & Zookeeper** for event streaming
- **ClickHouse** as sink
- Docker-based deployment

---

## Goal of Load Testing

- Simulate **high volumes of insert/update/delete** operations on MySQL.
- Monitor the impact on:
  - Kafka topic throughput
  - CDC latency (from MySQL to ClickHouse)
  - System resource usage (optional)
- Ensure **stability and performance** under increasing loads.

---

## Steps (Using JMeter to Load Test CDC)

### Introduction to Load Testing CDC Pipelines
- Overview of CDC architecture and JMeter’s role in simulating load on the **source system (MySQL)**.

---

### Launching JMeter and Test Plan Setup
- Start JMeter GUI or CLI mode.
- Create a new **Test Plan** and rename it to `CDC Load Test`.

---

### Adding the JDBC Driver to JMeter

1. Download the MySQL **JDBC Connector (JAR file)** from the official site:  
   👉 [MySQL JDBC Connector](https://dev.mysql.com/downloads/connector/j/)

2. Extract the downloaded archive to get the `.jar` file.

3. Navigate to your **JMeter installation folder**.

4. Open the `lib` directory inside the JMeter folder.

5. Paste the extracted `.jar` file into the `lib` directory.

---

### Configuring Thread Groups
- Add a **Thread Group** to simulate multiple virtual users.
  - Example:
    - Number of Threads: `50`
    - Ramp-Up Time: `10 seconds`
    - Loop Count: `100`

---

### Setting Up JDBC Connection to MySQL
- Add a **JDBC Connection Configuration** element.
  - JDBC Driver: `com.mysql.cj.jdbc.Driver`
  - URL: `jdbc:mysql://localhost:3306/testdb`
  - Username/Password
- Download MySQL JDBC driver and place it in `lib/` folder of JMeter.

---

### Adding JDBC Samplers (Insert/Update/Delete)
- Add multiple **JDBC Request** samplers to:
  - Insert records
  - Update existing rows
  - Delete some rows
- Use dynamic/random data using **CSV Data Set Config** or **JMeter Functions**.

---

### Validating Kafka Messages (Optional)
- Set up a Kafka consumer using a shell script, Kafka UI, or another tool.
- Confirm that events are published to the correct Kafka topics.

---

### Monitoring ClickHouse Sink
- Run ClickHouse queries to validate if the changes have been reflected.
- Optionally, create a custom JMeter sampler using **JDBC** to validate records in ClickHouse.

---

### Adding Listeners and Reports
- Add listeners to visualize test results:
  - View Results Tree
  - Aggregate Report
  - Summary Report
  - Response Time Graph

---

### Adding Assertions
- Validate that:
  - MySQL operations succeed (no error in response)
  - Kafka messages are produced (by checking offsets)
  - ClickHouse receives and reflects those records

---

### Duration and Performance Thresholds
- Add **Duration Assertion** and **Response Assertion** to check:
  - Time taken per JDBC operation
  - Max acceptable response time
  - Error % threshold

---

### Increasing Load for Stress Test
- Gradually increase:
  - Threads: 100 → 500
  - Loop Count
  - Number of operations per user
- Observe:
  - Kafka lag
  - Debezium connector delay
  - ClickHouse ingestion rate

---

### CLI Execution for Large Tests
- Run in non-GUI mode for better performance:
  ```bash
  jmeter -n -t cdc_load_test.jmx -l results.jtl -e -o report/
  ```
- Open `report/index.html` for the summary.
---

### Test Plan 
- You can use already provided test plan 
- [Mysql Load Testing](https://github.com/Datavolt/debezium-cdc/blob/main/TestPlan/MySqlTestPlan.jmx)

### References
- [Unleash The Power Of Performance Load Testing With Jmeter Tutorial](https://www.youtube.com/watch?v=CPSWuwm8CeU&list=PLVCgi5HZ0-Yt3N9tbjzi-WVQaH9UKkz73&index=1&ab_channel=VikasJha)
- [Apache JMeter | Load Testing | Performance Testing](https://www.youtube.com/watch?v=o5yJXBQo-XU&list=PLVCgi5HZ0-Yt3N9tbjzi-WVQaH9UKkz73&index=2&pp=iAQB)
- [Jmeter Variables and Counter](https://www.youtube.com/watch?v=vvETfo7_H24&list=PLVCgi5HZ0-Yt3N9tbjzi-WVQaH9UKkz73&index=3&ab_channel=VikasJha)
- [JMeter Database Load Testing](https://www.youtube.com/watch?v=6i4Qc_jR7LE&list=PLVCgi5HZ0-Yt3N9tbjzi-WVQaH9UKkz73&index=4&ab_channel=VikasJha)
- [Performing a Database Load Test on MySQL using Apache JMeter](https://anivaz.medium.com/performing-a-database-load-test-on-mysql-using-apache-jmeter-2ea761dbe3d5)
