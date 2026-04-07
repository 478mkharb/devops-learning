# 💰 Salary API – Production Setup Guide (Scylla + Redis + Migrations)

End-to-end, **repeatable** setup for the Salary microservice with **ScyllaDB (Cassandra-compatible)**, **Redis**, **Swagger**, and **Makefile-driven migrations**.

---

## 📌 Tech Stack

* Java 17 (OpenJDK)
* Spring Boot 3.x
* ScyllaDB (Cassandra protocol)
* Redis
* Maven Wrapper
* Swagger (springdoc-openapi)
* golang-migrate (for Cassandra migrations)

---

## 🚀 1. Prerequisites

```bash
sudo apt update
sudo apt install -y openjdk-17-jdk curl git make
java -version
```

---

## ⚙️ 2. Install ScyllaDB (official)

```bash
curl -sSf https://get.scylladb.com/server | sudo bash
sudo apt install -y scylla
```

### Configure (recommended answers)

```bash
sudo scylla_setup
```

* Kernel check: YES
* Auto start service: YES
* NTP: YES
* RAID/XFS: NO (VM)
* Coredumps: NO
* System config: NO
* IOTune: YES

---

## ▶️ 3. Start Scylla

```bash
sudo systemctl enable scylla-server
sudo systemctl start scylla-server
nodetool status
```

Expected: `UN 127.0.0.1`

---

## 🔴 4. Install Redis

```bash
sudo apt install -y redis-server
sudo systemctl enable redis
sudo systemctl start redis
redis-cli ping   # PONG
```

---

## 📦 5. Clone Project

```bash
git clone <your-repo-url>
cd salary-api
```

---

## 🗄️ 6. Migrations (golang-migrate)

### 6.1 Install migrate CLI

```bash
curl -L https://github.com/golang-migrate/migrate/releases/latest/download/migrate.linux-amd64.tar.gz | tar xvz
sudo mv migrate /usr/local/bin/
migrate -version
```

### 6.2 Verify migration files

`migration/000001_create_employee_salary_table.up.sql`

```sql
CREATE KEYSPACE IF NOT EXISTS salary_keyspace
WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1};

USE salary_keyspace;

CREATE TABLE IF NOT EXISTS employee_salary (
  id text PRIMARY KEY,
  name text,
  salary float,
  process_date text,
  status text
);
```

`migration/000001_create_employee_salary_table.down.sql`

```sql
DROP TABLE IF EXISTS salary_keyspace.employee_salary;
```

> Note: Cassandra/Scylla does **not** support `INSERT ... SELECT` or joins. Use migrations + application writes.

---

## 🧰 7. Makefile (migration + app)

Ensure your `Makefile` contains (TAB-indented commands):

```makefile
# CONFIG
CASSANDRA_HOST=127.0.0.1
CASSANDRA_PORT=9042
KEYSPACE=salary_keyspace

MIGRATION_PATH=./migration
DB_URL=cassandra://$(CASSANDRA_HOST):$(CASSANDRA_PORT)/$(KEYSPACE)

# MIGRATIONS
run-migrations:
	migrate -source file://$(MIGRATION_PATH) -database "$(DB_URL)" up

migrate-down:
	migrate -source file://$(MIGRATION_PATH) -database "$(DB_URL)" down

migrate-version:
	migrate -source file://$(MIGRATION_PATH) -database "$(DB_URL)" version

# APP
run:
	./mvnw spring-boot:run

build:
	./mvnw clean install
```

> ⚠️ Ensure keyspace is `salary_keyspace` (not `employee_db`).

---

## 🧪 8. Run Migrations

```bash
make run-migrations
```

Verify:

```bash
cqlsh -e "DESCRIBE KEYSPACES;"
cqlsh -e "USE salary_keyspace; DESCRIBE TABLES;"
```

Expected table: `employee_salary`

---

## ⚙️ 9. Application Configuration

`src/main/resources/application.yml`

```yaml
server:
  port: 8082

spring:
  data:
    cassandra:
      contact-points: 127.0.0.1
      port: 9042
      keyspace-name: salary_keyspace
      local-datacenter: datacenter1

    redis:
      host: 127.0.0.1
      port: 6379

management:
  endpoints:
    web:
      exposure:
        include: "*"
```

---

## 📄 10. Entity Mapping (CRITICAL)

`src/main/java/.../model/Employee.java`

```java
@Table("employee_salary")
public class Employee {

    @PrimaryKey
    private String id;

    private String name;
    private float salary;
    private String processDate;
    private String status;
}
```

> Table name **must match** migration (`employee_salary`). Types must align (float ↔ float).

---

## 📘 11. Swagger Configuration

`src/main/java/.../swagger/OpenAPIConfig.java`

```java
Server devServer = new Server();
devServer.setUrl("/"); // use same host/port as UI
```

---

## ▶️ 12. Run Application

```bash
make run
# or
./mvnw spring-boot:run
```

---

## 🧪 13. API Testing

### ➤ Create Record

```bash
curl -X POST http://localhost:8082/api/v1/salary/create/record \
-H "Content-Type: application/json" \
-d '{
  "id": "1",
  "name": "Mukesh",
  "salary": 50000,
  "processDate": "2026-04-07",
  "status": "SUCCESS"
}'
```

### ➤ Get All

```bash
curl http://localhost:8082/api/v1/salary/search/all
```

---

## 📊 14. Swagger UI

```text
http://<HOST>:8082/swagger-ui/index.html
```

* Click endpoint → **Try it out** → **Execute**
* With `devServer.setUrl("/")`, Swagger uses the same host/port

---

## 🛠️ Troubleshooting

### ❌ 500 on POST

* Table mismatch (`@Table` vs DB)
* Column/type mismatch

### ❌ Swagger calls wrong port

* Ensure `devServer.setUrl("/")`

### ❌ Migration fails (keyspace)

* Use `salary_keyspace` (not `employee_db`)

### ❌ `make run-migrations` not found

* Add target in Makefile
* Ensure TAB indentation

### ❌ `Permission denied` running `.sql`

* Use `make run-migrations` or `cqlsh -f <file>`

---

## 🧱 Architecture

```text
Client → Salary API (8082)
                 → ScyllaDB (9042)
                 → Redis (6379)
```

---

## 🏁 Status Checklist

* [x] Scylla installed & running
* [x] Redis installed & running
* [x] Migrations automated via Makefile
* [x] API running on 8082
* [x] Swagger working with correct base URL
* [x] Data persists in `employee_salary`

---
