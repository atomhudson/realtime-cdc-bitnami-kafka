# Detail overview of ```docker-compose-mysql.yml```
```yaml 
version: '3.8'

services:
  mysql:
    image: mysql:8.0
    container_name: mysql
    restart: unless-stopped
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_USER: debezium
      MYSQL_PASSWORD: dbz
      MYSQL_DATABASE: inventory
    command:
      --server-id=223344 \
      --log-bin=mysql-bin \
      --binlog-format=ROW \
      --binlog-row-image=FULL \
      --gtid-mode=ON \
      --enforce-gtid-consistency=TRUE \
      --log-slave-updates=TRUE \
      --plugin-load=mysql_native_password.so
    volumes:
      - mysql-data:/var/lib/mysql

volumes:
  mysql-data:
```
---

## Why this part is written:
```yaml
ports:
  - "3306:3306"
environment:
  MYSQL_ROOT_PASSWORD: root
  MYSQL_USER: debezium
  MYSQL_PASSWORD: dbz
  MYSQL_DATABASE: inventory
```

---

### `ports: - "3306:3306"`

This maps ports between your **host machine** and the **MySQL container**.

```plaintext
HOST_PORT : CONTAINER_PORT
```

#### `"3306:3306"` means:
- Expose MySQL running inside the container on port `3306` (default for MySQL)
- Make it accessible from your local machine on port `3306`

So you can connect using:
```bash
mysql -h localhost -P 3306 -u root -p
```

---

### What if I use `"3307:3306"` instead?

```yaml
ports:
  - "3307:3306"
```

It means:
- MySQL still runs **inside the container** on its default port `3306`
- But from **outside the container (on your host machine)**, you connect using **port `3307`**

You’d then connect like:
```bash
mysql -h localhost -P 3307 -u root -p
```

Useful when:
- You have **another MySQL** running on your machine's `3306`
- Or you run **multiple MySQL containers** and want to expose them on different ports

---

### What happens if I write `"3307:3307"`?

```yaml
ports:
  - "3307:3307"
```

This assumes that **MySQL is listening inside the container on port 3307**, but by default it’s listening on `3306`.

So unless you **change MySQL config to also use port 3307 inside**, this will **not work** and you'll get connection errors.

---

## Why these environment variables?

These set up MySQL with secure defaults at container startup:

| Variable              | Purpose                                                  |
|-----------------------|----------------------------------------------------------|
| `MYSQL_ROOT_PASSWORD` | Sets password for the **root** (admin) user              |
| `MYSQL_USER`          | Creates a new user (e.g., `debezium`)                    |
| `MYSQL_PASSWORD`      | Password for that user                                   |
| `MYSQL_DATABASE`      | Creates a database with that name (e.g., `inventory`)    |

So when the container starts:
- A database `inventory` is created
- A user `debezium` with password `dbz` is created
- That user is **granted access** to the `inventory` DB

This saves you from manually doing all this after startup!

---

### Summary

| What you write         | What it means                                           |
|------------------------|---------------------------------------------------------|
| `3306:3306`            | Default: Connect from host on 3306 → container 3306     |
| `3307:3306`            | Connect from host on 3307 → container still uses 3306   |
| `3307:3307`            | Only works if MySQL **inside** is reconfigured to 3307  |
| `environment` section  | Auto-creates DB, users, and sets passwords              |

---

## Why command is written: 

---

### 1. `--server-id=223344`
- Every MySQL server in a replication setup needs a **unique server ID**.
- Required for binary log-based replication (used by Debezium).

---

### 2. `--log-bin=mysql-bin`
- Enables the **binary log**, which records all changes to the database (INSERTs, UPDATEs, DELETEs).
- **Debezium reads from this binlog** to detect changes.

---

### 3. `--binlog-format=ROW`
- Controls how changes are written to the binlog.
- `ROW` format logs the actual **row-level changes**, which is essential for Debezium (unlike `STATEMENT` or `MIXED`).

---

### 4. `--binlog-row-image=FULL`
- Ensures **entire row data** (both before and after change) is written to the binlog.
- Important for getting full context during updates (e.g., for auditing or rollback use cases).

---

### 5. `--gtid-mode=ON`

**What it does (in simple terms):**
- Enables **GTIDs** (Global Transaction Identifiers).
- A **GTID** is a unique ID that MySQL gives to every transaction.
- Makes replication easier, more reliable, and cleaner.

#### Think of it like:
Every transaction gets a **unique serial number**—like a receipt. So even if it moves from one database to another, you always know **what happened**, **when**, and **in what order**.

#### Why it’s helpful:
When syncing MySQL with another system (like Debezium or a replica), GTIDs make it easier to:
- Track **exactly which changes** have already been processed.
- **Resume replication** smoothly after failures.
- Avoid applying the same transaction more than once.

#### Example:
```sql
-- A GTID looks like this:
22334456-aaaa-bbbb-cccc-1234567890ab:1
```

So if Debezium stops and restarts, it can say:
> “Last GTID I processed was 22334456-...:1, so I’ll start from 2.”

---

### 6. `--enforce-gtid-consistency=TRUE`

**What it does:**
- Makes sure all your transactions are written in a way that’s **compatible with GTID**.
- Prevents certain non-GTID-friendly operations.

#### Why it’s important:
Some SQL operations (like temporary tables or certain `CREATE ... SELECT` queries) **can’t be tracked properly** with GTIDs. If you don’t enforce consistency, those transactions could cause replication to **break** or **become unreliable**.

By enabling this, MySQL **throws an error** if you try to run such queries—so you’re always writing GTID-safe code.

#### Example:
Without this option:
```sql
CREATE TEMPORARY TABLE temp AS SELECT * FROM users;
-- MySQL would allow it, but GTID might get confused.
```

With this option:
```bash
Error: This operation is not allowed when enforce-gtid-consistency=TRUE
```

So it **protects your replication setup** from breaking.

---

### 7. `--log-slave-updates=TRUE`
- If this server were to act as a **replica**, it would also log changes it receives from its master.
- Ensures the **binlog contains a full history** even on replicas.

---

### 8. `--plugin-load=mysql_native_password.so`
**What it does:**
- Tells MySQL to **load the old authentication plugin** called `mysql_native_password`.

#### Why it’s needed:
Newer MySQL versions use a more secure default plugin called `caching_sha2_password`.  
But tools like **Debezium**, some **older drivers**, and even **some JDBC clients** don’t work well with it.

To prevent login/auth issues, you use this option to make sure:
- The older, widely compatible `mysql_native_password` is available.
- You can still connect from tools like Debezium, JDBC, Postman, etc.

#### Example:
If this is **not enabled**, you might get errors like:
```
Client does not support authentication protocol requested by server
```

But with this plugin loaded, and your user set like this:
```sql
ALTER USER 'debezium'@'%' IDENTIFIED WITH mysql_native_password BY 'dbz';
```

Everything works fine. Debezium can connect and read the binlog.

---

## Why ```version : mysql-data: ``` is written ?

---

### `volumes: mysql-data:` — Why do we write this in `docker-compose-mysql.yml`?

This line is used to **persist MySQL data** even if the container is stopped, removed, or recreated.

---

### Think of it like this:

When you use Docker, by default, all the files inside a container are **temporary**. If you restart or delete the container, **everything is gone**, including your database!

So we use **volumes** to:
- **Store data outside the container**
- **Keep MySQL data safe** between restarts
- **Avoid losing tables, rows, and schema**

---

### Where it's used in the file:

```yaml
services:
  mysql:
    ...
    volumes:
      - mysql-data:/var/lib/mysql
```

This line tells Docker:
> “Please store MySQL’s data (normally found in `/var/lib/mysql`) in a named volume called `mysql-data`.”

Then at the bottom:

```yaml
volumes:
  mysql-data:
```

This **defines** the volume so Docker knows to create it if it doesn’t exist.

---

### What’s inside `/var/lib/mysql`?
That’s where MySQL stores:
- All your **databases**
- Tables
- Indexes
- Binlogs (for CDC)
- Metadata

---

### Real-world example:

1. You run your container, create a database, and add some data.
2. Then you stop and delete the container.
3. Next time you `docker-compose up`, Docker **reattaches the same `mysql-data` volume**.
4. Your data is still there 

---

### ✅ Without volume:  
You lose **everything** on container restart.

### ✅ With volume:  
You keep all your data and logs.

---
