# Salary API

---

## 📌 Overview

The **Salary API** is a Spring Boot–based microservice responsible for managing employee salary records. It integrates with:

* **ScyllaDB (Cassandra-compatible)** → persistent storage
* **Redis** → caching layer
* **Swagger (OpenAPI)** → API documentation

This document provides a **step-by-step, production-ready setup** with detailed explanations of **what** and **why** for every step.

---

# 🧱 Architecture

```
Client → Salary API (Spring Boot)
                  ↓
          ScyllaDB (Cassandra)
                  ↓
               Redis
```

---

# 🛠️ Tech Stack

| Component       | Purpose                    |
| --------------- | -------------------------- |
| Java 17         | Runtime environment        |
| Spring Boot 3.x | Microservice framework     |
| ScyllaDB        | Distributed NoSQL database |
| Redis           | Caching layer              |
| Maven Wrapper   | Build automation           |
| Swagger         | API documentation          |
| golang-migrate  | DB schema management       |

---

# 📁 Directory Structure

```
salary-api/
├── src/
│   ├── main/
│   │   ├── java/com/opstree/microservice/salary/
│   │   │   ├── controller/        # REST endpoints
│   │   │   ├── service/           # Business logic
│   │   │   ├── repository/        # DB interaction
│   │   │   ├── model/             # Entity classes
│   │   │   ├── config/            # Configuration classes
│   │   │   └── swagger/           # Swagger config
│   │   └── resources/
│   │       └── application.yml    # App configuration
│
├── migration/                     # Cassandra schema scripts
├── Makefile                      # Automation commands
├── pom.xml                       # Dependencies
├── mvnw                          # Maven wrapper
└── README.md
```

---

# 🚀 STEP 1 — System Preparation

```bash
sudo apt update
sudo apt install -y openjdk-17-jdk curl git make
java -version
```

### ✅ WHY?

* Java 17 is required by Spring Boot 3.x
* `curl`, `git`, `make` are required for automation and setup
* Ensures a clean base environment

---

# 🧠 STEP 2 — Install ScyllaDB

```bash
curl -sSf https://get.scylladb.com/server | sudo bash
sudo apt install -y scylla
```

### Configure:

```bash
sudo scylla_setup
```

### ✅ WHY?

* ScyllaDB is a high-performance Cassandra-compatible database
* Required for storing salary records
* `scylla_setup` optimizes system parameters

---

# ▶️ STEP 3 — Start ScyllaDB

```bash
sudo systemctl enable scylla-server
sudo systemctl start scylla-server
nodetool status
```

### Expected:

```
UN 127.0.0.1
```

### ✅ WHY?

* Ensures DB is running and reachable
* `UN` = Up & Normal state

---

# 🔴 STEP 4 — Install Redis

```bash
sudo apt install -y redis-server
sudo systemctl enable redis
sudo systemctl start redis
redis-cli ping
```

### Expected:

```
PONG
```

### ✅ WHY?

* Redis improves performance via caching
* Used by Spring Boot for fast data retrieval

---

# 📦 STEP 5 — Clone Project

```bash
git clone <your-repo-url>
cd salary-api
```

### ✅ WHY?

* Fetches project codebase
* Ensures consistent structure across environments

---

# 🧩 STEP 6 — Database Migration Setup

## Install migrate CLI

```bash
curl -L https://github.com/golang-migrate/migrate/releases/latest/download/migrate.linux-amd64.tar.gz | tar xvz
sudo mv migrate /usr/local/bin/
migrate -version
```

### ✅ WHY?

* Cassandra does NOT support schema evolution automatically
* Migrations ensure **repeatable and version-controlled DB setup**

---

# 🧱 STEP 7 — Schema Definition

Migration file creates:

```sql
CREATE KEYSPACE salary_keyspace;
CREATE TABLE employee_salary (...);
```

### ✅ WHY?

* Keyspace = logical DB in Cassandra
* Table defines salary structure
* Required before application startup

---

# ⚙️ STEP 8 — Run Migrations

```bash
make run-migrations
```

### Verify:

```bash
cqlsh -e "USE salary_keyspace; DESCRIBE TABLES;"
```

### ✅ WHY?

* Ensures DB schema exists
* Prevents runtime failures

---

# ⚙️ STEP 9 — Application Configuration

```bash
nano ~/salary-api/src/main/resources/application.yml
```
Change 
```yaml
server:
  port: 8082

spring:
  cassandra:
    contact-points: 127.0.0.1
    port: 9042
    keyspace-name: salary_keyspace
    local-datacenter: datacenter1

  data:
    redis:
      host: 127.0.0.1
      port: 6379
```

### ✅ WHY?

* `spring.cassandra` → required by driver (CRITICAL)
* `local-datacenter` → mandatory for Cassandra driver
* Redis config → enables caching

---

# 🧬 STEP 10 — Entity Mapping
```bash
nano OT-Microservices/salary-api/src/main/java/com/opstree/microservice/salary/model/Employee.java
```
Check 
```java
@Table("employee_salary")
public class Employee {
    @PrimaryKey
    private String id;
}
```

### ✅ WHY?

* Maps Java object → Cassandra table
* Table name must match migration exactly
* Ensures ORM layer works correctly

---

# 📘 STEP 11 — Swagger Configuration
```bash
nano ~/salary-api/src/main/java/com/opstree/microservice/salary/config/OpenAPIConfig.java
```
Change 
```java
devServer.setUrl("http://localhost:8080");
```
to 
```java
devServer.setUrl("/");
```

### ✅ WHY?

* Ensures Swagger uses same host/port
* Prevents wrong API calls (8080 vs 8082 issue)

---

# ▶️ STEP 12 — Run Application

```bash
./mvnw spring-boot:run
```

### OR

```bash
make run
```

### ✅ WHY?

* Starts embedded Tomcat server
* Initializes all beans and DB connections

---

# 🔍 STEP 13 — Health Check

```bash
curl http://localhost:8082/actuator/health
```

### ✅ WHY?

* Verifies:

  * Cassandra connection
  * Redis connection
  * App health

---

# 🧪 STEP 14 — API Testing

## Create Record

```bash
curl -X POST http://localhost:8082/api/v1/salary/create/record \
-H "Content-Type: application/json" \
-d '{
  "id": "1",
  "name": "Mukesh",
  "salary": 50000.0,
  "processDate": "2026-04-08",
  "status": "SUCCESS"
}'
```

## Fetch Records

```bash
curl http://localhost:8082/api/v1/salary/search/all
```

### ✅ WHY?

* Confirms end-to-end flow:

  * API → Service → DB → Response

---

# 📊 STEP 15 — Swagger UI

```
http://localhost:8082/salary-documentation
```

### ✅ WHY?

* Interactive API testing
* Developer-friendly interface

---

# 🛠️ Troubleshooting

### ❌ Cassandra connection fails

* Check `local-datacenter`

### ❌ Migration error

* Ensure keyspace = `salary_keyspace`

### ❌ Swagger wrong port

* Ensure:

```java
devServer.setUrl("/");
```

### ❌ Build failure

```bash
./mvnw clean install -DskipTests
```

---

# 🏁 Final Validation Checklist

* [x] ScyllaDB running
* [x] Redis running
* [x] Migration successful
* [x] App running on 8082
* [x] Health check UP
* [x] POST API working
* [x] GET API working
* [x] Swagger working

---

# 🎯 Summary

This setup ensures:

* Scalable NoSQL storage (ScyllaDB)
* High-performance caching (Redis)
* Clean architecture (Spring Boot)
* Repeatable deployments (Makefile + migrations)

---

🔥 Salary API is now **production-ready and integration-safe**
