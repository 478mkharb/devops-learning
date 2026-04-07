# Employee API – Production-Grade Setup SOP (EC2 Ubuntu 22.04 | No Docker)

---

## 1. Objective

This document provides a **step-by-step Standard Operating Procedure (SOP)** to:

* Provision a fresh Ubuntu EC2 instance
* Install all required dependencies
* Configure ScyllaDB
* Execute database migrations using golang-migrate
* Run and validate the Employee API

This guide is **reproducible, DevOps-aligned, and production-oriented**.

---

## 2. Architecture

```
Client → Go (Gin) API → Business Logic → ScyllaDB
                                     → Redis (optional)
```

---

## 3. Project Structure (Detailed Explanation)

### Root Level

| Component       | Description                                |
| --------------- | ------------------------------------------ |
| main.go         | Application entry point                    |
| config.yaml     | Runtime configuration (DB, Redis)          |
| Makefile        | Build, migration, and execution automation |
| go.mod / go.sum | Dependency management                      |
| migration/      | Database schema (SQL migrations)           |
| migration.json  | Migration tool configuration               |
| docs/           | Swagger API documentation                  |
| static/         | Static assets                              |
| Dockerfile      | Containerization (future use)              |

---

### api/

Contains core API handlers.

| File      | Purpose                 |
| --------- | ----------------------- |
| api.go    | Business logic handlers |
| health.go | Health check endpoint   |

---

### client/

Handles external integrations.

| File        | Purpose             |
| ----------- | ------------------- |
| scylladb.go | ScyllaDB connection |
| redis.go    | Redis connection    |

---

### config/

| File     | Purpose                       |
| -------- | ----------------------------- |
| viper.go | Loads config.yaml using Viper |

---

### middleware/

| File       | Purpose                  |
| ---------- | ------------------------ |
| logging.go | Request/response logging |

---

### model/

| File        | Purpose               |
| ----------- | --------------------- |
| employee.go | Employee schema       |
| health.go   | Health response model |
| config.go   | Config struct         |

---

### routes/

| File      | Purpose            |
| --------- | ------------------ |
| routes.go | Route registration |

---

### migration/

| File       | Purpose         |
| ---------- | --------------- |
| *.up.sql   | Schema creation |
| *.down.sql | Rollback schema |

This project uses **golang-migrate CLI**, not manual cqlsh execution.

---

## 4. EC2 Setup

### Step 1 — Launch Instance

* OS: Ubuntu 22.04
* Instance: t2.micro
* Open ports:

  * 22 (SSH)
  * 8080 (Application)

---

### Step 2 — Connect

```bash
ssh ubuntu@<EC2-PUBLIC-IP>
```

---

### Step 3 — System Update

```bash
sudo apt update && sudo apt upgrade -y
```

---

### Step 4 — Install Base Packages

```bash
sudo apt install git curl wget unzip jq -y
```

---

## 5. Install Go

```bash
sudo apt install golang-go -y
```

Verify:

```bash
go version
```

---

## 6. Install ScyllaDB

```bash
curl -sSf https://get.scylladb.com/server | sudo bash
sudo apt install scylla -y
```

---

## 7. Configure ScyllaDB

```bash
sudo scylla_setup
```

Recommended:

* IOTune → YES
* Dedicated machine → NO

---

### Enable Developer Mode (Mandatory for low RAM)

```bash
sudo nano /etc/scylla/scylla.yaml
```

Update:

```yaml
developer_mode: true
```

Restart:

```bash
sudo systemctl restart scylla-server
sudo systemctl status scylla-server
```

---

## 8. Database Initialization

```bash
cqlsh
```

```sql
CREATE KEYSPACE employee WITH replication = {
  'class': 'SimpleStrategy',
  'replication_factor': 1
};
```

Exit:

```bash
exit
```

---

## 9. Clone Repository

```bash
git clone https://github.com/OT-MICROSERVICES/employee-api.git
cd employee-api
```

---

## 10. Install Dependencies

```bash
go mod tidy
```

---

## 11. Configure Migration (CRITICAL STEP)

Edit:

```bash
nano migration.json
```

### Correct Configuration:

```json
{
  "database": "cassandra://127.0.0.1:9042/employee"
}
```

Notes:

* Do NOT use Docker IP (172.x.x.x)
* Always use localhost for local setup

---

## 12. Install golang-migrate CLI

```bash
curl -L https://github.com/golang-migrate/migrate/releases/latest/download/migrate.linux-amd64.tar.gz | tar xvz
sudo mv migrate /usr/local/bin/
```

Verify:

```bash
migrate -version
```

---

## 13. Run Database Migration

```bash
make run-migrations
```

This executes:

```bash
migrate -source file://migration -database <connection-string> up
```

---

## 14. Configure Application

```bash
nano config.yaml
```

```yaml
scylladb:
  host: 127.0.0.1
  port: 9042
  keyspace: employee

redis:
  enabled: false
```

---

## 15. Run Application

```bash
go run main.go
```

Expected:

```
Listening and serving HTTP on :8080
```

---

## 16. API Validation

### Health Check

```bash
curl http://localhost:8080/api/v1/employee/health
```

---

### Create Employee

```bash
curl -X POST http://localhost:8080/api/v1/employee/create \
-H "Content-Type: application/json" \
-d '{
  "id": "1",
  "name": "Mukesh",
  "designation": "DevOps",
  "department": "Engineering",
  "joining_date": "2026-04-05",
  "address": "Delhi",
  "office_location": "Delhi",
  "status": "Active",
  "email": "mukesh@test.com",
  "phone_number": "9999999999"
}'
```

---

### Fetch Employee

```bash
curl "http://localhost:8080/api/v1/employee/search?id=1"
```

---

## 17. Swagger Documentation

After starting the application:

```
http://localhost:8080/swagger/index.html
```

---

## 18. Common Issues and Fixes

### Issue: migrate connection timeout

Cause: Wrong IP (Docker IP used)

Fix:

```json
127.0.0.1 instead of 172.x.x.x
```

---

### Issue: go command not found

```bash
sudo apt install golang-go -y
```

---

### Issue: Scylla not starting

```bash
journalctl -u scylla-server
```

Ensure:

```yaml
developer_mode: true
```

---

### Issue: Keyspace does not exist

```sql
CREATE KEYSPACE employee ...
```

---

## 19. DevOps Mapping

| Layer      | Tool           |
| ---------- | -------------- |
| Build      | Go modules     |
| Migration  | golang-migrate |
| Config     | YAML + Viper   |
| DB         | ScyllaDB       |
| Automation | Makefile       |

---

## 20. Future Enhancements

* systemd service
* NGINX reverse proxy
* Redis enablement
* CI/CD pipeline (Jenkins)
* Infrastructure automation (Terraform)

---

## 21. Summary

```
Go + Gin + ScyllaDB + golang-migrate + Makefile
```

This setup is fully aligned with real-world DevOps workflows and is reproducible across environments.

---
