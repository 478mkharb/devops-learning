# 🚀 Employee API — Production Setup Guide (EC2 | Ubuntu 22)

---

## 📌 Overview

This document provides a **step-by-step, production-grade setup** of the Employee API from scratch on a **fresh EC2 Ubuntu 22 instance**, following the original project structure strictly.

It includes:

* Infrastructure setup
* Dependency installation
* Database configuration (ScyllaDB)
* Cache configuration (Redis)
* Migration execution
* Swagger setup
* API validation

Each step includes **WHY it is required**.

---

# 🏗️ Architecture

```
Client → Gin (Go API)
            ↓
        ScyllaDB (Primary DB)
            ↓
        Redis (Cache Layer)
            ↓
        Swagger (API Docs)
```

---

# 📁 Directory Structure (Important)

```
employee-api/
├── api/                # Business logic handlers
├── client/             # DB & Redis connections
├── config/             # Config loader (Viper)
├── middleware/         # Logging middleware
├── model/              # Data models
├── routes/             # Route registration
├── migration/          # DB schema
├── docs/               # Swagger docs
├── main.go             # Entry point
├── config.yaml         # Runtime config
├── migration.json      # Migration DB config
├── Makefile            # Automation
```

---

# ⚙️ STEP 1 — Launch EC2 Instance

### Configuration:

* OS: Ubuntu 22.04
* Instance: t2.micro
* Ports:

  * 22 (SSH)
  * 8080 (API)

### WHY?

* EC2 provides isolated environment
* Port 8080 is used by Gin server

---

# 🔐 STEP 2 — Connect to Server

```bash
ssh ubuntu@<EC2-IP>
```

---

# 🔄 STEP 3 — System Update

```bash
sudo apt update && sudo apt upgrade -y
```

### WHY?

* Ensures latest security patches
* Prevents dependency conflicts

---

# 📦 STEP 4 — Install Base Packages

```bash
sudo apt install -y git curl wget unzip jq
```

### WHY?

* git → clone repo
* curl/wget → download binaries
* jq → JSON parsing/debugging

---

# 🟢 STEP 5 — Install Go (Runtime)

```bash
wget https://go.dev/dl/go1.22.5.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.22.5.linux-amd64.tar.gz

echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
source ~/.bashrc
```

### Verify:

```bash
go version
```

### WHY?

* Required to compile & run API
* Version compatibility avoids Swagger/tooling issues

---

# 🧠 STEP 6 — Install ScyllaDB (Database)

```bash
curl -sSf https://get.scylladb.com/server | sudo bash
sudo apt install -y scylla
```

### Configure:

```bash
sudo scylla_setup
```

### Enable dev mode:

```bash
sudo nano /etc/scylla/scylla.yaml
```

```yaml
developer_mode: true
```

```bash
sudo systemctl restart scylla-server
sudo systemctl enable scylla-server
```

### Verify:

```bash
nodetool status
```

### WHY?

* Primary DB for employee data
* Cassandra-compatible scalable DB

---

# 🗄️ STEP 7 — Create Keyspace

```bash
cqlsh
```

```sql
CREATE KEYSPACE employee WITH replication = {
  'class': 'SimpleStrategy',
  'replication_factor': 1
};
```

### WHY?

* Logical database container
* Required before migrations

---

# 📥 STEP 8 — Clone Repository

```bash
git clone https://github.com/OT-MICROSERVICES/employee-api.git
cd employee-api
```

---

# 🔧 STEP 9 — Install Dependencies

```bash
go mod tidy
```

### WHY?

* Resolves all Go dependencies
* Ensures build reproducibility

---

# 🧰 STEP 10 — Install Migration Tool

```bash
curl -L https://github.com/golang-migrate/migrate/releases/latest/download/migrate.linux-amd64.tar.gz | tar xvz
sudo mv migrate /usr/local/bin/
```

### WHY?

* Executes SQL schema files
* Version control for DB schema

---

# ⚙️ STEP 11 — Configure Migration

```bash
nano migration.json
```

```json
{
  "database": "cassandra://127.0.0.1:9042/employee"
}
```

### WHY?

* Connects migration tool to DB

---

# 🧱 STEP 12 — Run Migrations

```bash
make run-migrations
```

### Verify:

```bash
cqlsh
USE employee;
DESCRIBE TABLES;
```

### WHY?

* Creates required tables
* Without this API fails at runtime

---

# ⚡ STEP 13 — Install Redis (Cache)

```bash
sudo apt install -y redis-server
sudo systemctl start redis
sudo systemctl enable redis
```

### Verify:

```bash
redis-cli ping
```

### WHY?

* Improves performance via caching
* Reduces DB load

---

# ⚙️ STEP 14 — Configure Redis

Edit config.yaml:

```yaml
redis:
  enabled: true
  host: "127.0.0.1"
  port: 6379
  password: ""
  database: 0
```

### WHY?

* Enables cache layer
* Without this Redis is unused

---

# 📚 STEP 15 — Setup Swagger

```bash
go install github.com/swaggo/swag/cmd/swag@v1.8.12

echo 'export PATH=$PATH:$(go env GOPATH)/bin' >> ~/.bashrc
source ~/.bashrc
```

```bash
rm -rf docs/*
make swagger
```

### FIX (Important)

Edit main.go:

```go
url := ginSwagger.URL("/swagger/doc.json")
```

### WHY?

* Enables API documentation UI
* Fixes CORS issues

---

# ▶️ STEP 16 — Run Application

```bash
go run main.go
```

---

# ✅ STEP 17 — API Validation

### Health Check

```bash
curl http://localhost:8080/api/v1/employee/health
```

### Expected:

```json
{"message":"Employee API is up and running"}
```

---

# 🌐 STEP 18 — Swagger UI

```
http://<EC2-IP>:8080/swagger/index.html
```

---

# 🧪 STEP 19 — Test API

### Create Employee

```bash
curl -X POST http://localhost:8080/api/v1/employee/create \
-H "Content-Type: application/json" \
-d '{
  "id": "1",
  "name": "Mukesh"
}'
```

---

# 🔍 Troubleshooting

### Port already in use

```bash
fuser -k 8080/tcp
```

### Swagger CORS issue

* Use relative path `/swagger/doc.json`

### Migration failure

* Check keyspace name

---

# 🏁 Final Checklist

* [x] Go installed
* [x] ScyllaDB running
* [x] Keyspace created
* [x] Migration successful
* [x] Redis running
* [x] Swagger working
* [x] API responding

---

# 🎯 Conclusion

You now have a **production-ready Employee API** with:

* Scalable DB (ScyllaDB)
* Optional cache (Redis)
* Fully documented API (Swagger)
* Clean modular architecture

---

🔥 End of README
