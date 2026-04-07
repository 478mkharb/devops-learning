# 💰 Salary API – Production Setup Guide

A complete step-by-step guide to install, configure, and run the **Salary Microservice** using **Spring Boot + ScyllaDB (Cassandra-compatible) + Redis + Swagger**.

---

# 📌 Tech Stack

* Java 17
* Spring Boot 3.x
* ScyllaDB (Cassandra compatible)
* Redis
* Maven Wrapper
* Swagger (SpringDoc OpenAPI)

---

# 🚀 1. Prerequisites

```bash
sudo apt update
sudo apt install -y openjdk-17-jdk curl git
```

Verify:

```bash
java -version
```

---

# ⚙️ 2. Install ScyllaDB (Official Script)

```bash
curl -sSf https://get.scylladb.com/server | sudo bash
sudo apt install scylla -y
```

---

# 🔧 3. Setup ScyllaDB

```bash
sudo scylla_setup
```

### Recommended choices:

| Option             | Selection   |
| ------------------ | ----------- |
| Kernel Check       | YES         |
| Service Auto Start | YES         |
| NTP                | YES         |
| RAID/XFS           | NO (for VM) |
| Coredumps          | NO          |
| System Config      | NO          |
| IOTune             | YES         |

---

# ▶️ 4. Start Scylla

```bash
sudo systemctl start scylla-server
sudo systemctl enable scylla-server
```

Check status:

```bash
nodetool status
```

Expected:

```text
UN 127.0.0.1
```

---

# 🗄️ 5. Setup Database

```bash
cqlsh
```

```sql
CREATE KEYSPACE salary_keyspace
WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1};

USE salary_keyspace;

CREATE TABLE employee_salary (
  id text PRIMARY KEY,
  name text,
  salary float,
  process_date text,
  status text
);
```

---

# 🔴 6. Install Redis

```bash
sudo apt install redis-server -y
sudo systemctl start redis
sudo systemctl enable redis
```

Verify:

```bash
redis-cli ping
```

Expected:

```text
PONG
```

---

# 📦 7. Clone Project

```bash
git clone <your-repo-url>
cd salary-api
```

---

# ⚙️ 8. Configure application.yml

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

# 📄 9. Fix Entity Mapping

Ensure file:

```bash
src/main/java/.../model/Employee.java
```

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

---

# 📘 10. Fix Swagger Configuration

File:

```bash
src/main/java/.../swagger/OpenAPIConfig.java
```

```java
Server devServer = new Server();
devServer.setUrl("/");
```

---

# ▶️ 11. Run Application

```bash
./mvnw clean
./mvnw spring-boot:run
```

---

# 🧪 12. Test APIs

## ➤ Create Record

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

## ➤ Get All

```bash
curl http://localhost:8082/api/v1/salary/search/all
```

---

# 📊 13. Swagger UI

Open:

```text
http://<IP>:8082/swagger-ui/index.html
```

---

# 🛠️ Troubleshooting

## ❌ 500 Error

* Table mismatch
* Wrong data types

## ❌ Swagger calling 8080

Fix:

```java
devServer.setUrl("/");
```

## ❌ Empty API response

* Data inserted in wrong table

---

# 📌 Final Architecture

```text
Client → Salary API → ScyllaDB
                 → Redis
```

---

# 🏁 Status

✔ Scylla Installed
✔ Redis Installed
✔ API Running
✔ Swagger Working
✔ Data Persisting

---

# 🚀 Next Steps

* Integrate with employee-api
* Add API Gateway
* Dockerize services
* Deploy on Kubernetes

---

# 👨‍💻 Author

Mukesh Kharb
