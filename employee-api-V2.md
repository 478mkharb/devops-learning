# Employee API

---

## 1. Objective

* Provision a fresh Ubuntu EC2 instance
* Install all required dependencies
* Configure ScyllaDB
* Execute database migrations using golang-migrate
* Generate and configure Swagger documentation
* Run and validate the Employee API

---

## 2. Architecture

```
Client → Go (Gin) API → Business Logic → ScyllaDB
                                     → Redis (optional)
                                     → Swagger Docs
```

---

## 3. Project Structure

### Root Level

| Component       | Description                           |
| --------------- | ------------------------------------- |
| main.go         | Application entry point               |
| config.yaml     | Runtime configuration (DB, Redis)     |
| Makefile        | Build, migration, swagger automation  |
| go.mod / go.sum | Dependency management                 |
| migration/      | Database schema (SQL migrations)      |
| migration.json  | Migration tool configuration          |
| docs/           | Swagger generated files (DO NOT EDIT) |
| static/         | Static assets                         |
| Dockerfile      | Containerization (future use)         |

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

Uses golang-migrate CLI.

---

## 4. EC2 Setup

### Step 1 — Launch Instance

* Ubuntu 22.04
* Open ports: 22, 8080

---

### Step 2 — Connect

```bash
ssh ubuntu@<EC2-IP>
```

---

### Step 3 — System Update

```bash
sudo apt update && sudo apt upgrade -y
```

---

### Step 4 — Base Packages

```bash
sudo apt install git curl wget unzip jq -y
```

---

## 5. Install Go (IMPORTANT)

```bash
wget https://go.dev/dl/go1.22.5.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.22.5.linux-amd64.tar.gz
```

```bash
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
source ~/.bashrc
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

Enable dev mode:

```bash
sudo nano /etc/scylla/scylla.yaml
```

```yaml
developer_mode: true
```

```bash
sudo systemctl restart scylla-server
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

---

## 9. Clone Repo

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

## 11. Migration Setup

```bash
nano migration.json
```

```json
{
  "database": "cassandra://127.0.0.1:9042/employee"
}
```

---

## 12. Install migrate CLI

```bash
curl -L https://github.com/golang-migrate/migrate/releases/latest/download/migrate.linux-amd64.tar.gz | tar xvz
sudo mv migrate /usr/local/bin/
```

---

## 13. Run Migration

```bash
make run-migrations
```

---

## 14. Swagger Setup (CRITICAL)

### Install swag (compatible version)

```bash
go install github.com/swaggo/swag/cmd/swag@v1.8.12
```

```bash
echo 'export PATH=$PATH:$(go env GOPATH)/bin' >> ~/.bashrc
source ~/.bashrc
```

---

### Generate Docs

```bash
rm -rf docs/*
make swagger
```

---

### Fix CORS Issue

Edit Swagger route:

```go
r.GET("/swagger/*any", ginSwagger.WrapHandler(
    swaggerFiles.Handler,
    ginSwagger.URL("/swagger/doc.json"),
))
```

---

## 15. Run Application

```bash
go run main.go
```

---

## 16. API Testing

```bash
curl http://localhost:8080/api/v1/employee/health
```

---

## 17. Swagger Access

```
http://<EC2-IP>:8080/swagger/index.html
```

---

## 18. Common Issues

### Swagger LeftDelim Error

Cause: Version mismatch

Fix:

```bash
go install swag@v1.8.12
```

---

### Swagger localhost issue

Fix: Use relative path `/swagger/doc.json`

---

### Migration timeout

Use localhost instead of Docker IP

---

### Go not found

Fix PATH and extraction

---

## 19. Final State

```
Go Installed
ScyllaDB Running
Migration Working
Swagger Working
API Functional
```
