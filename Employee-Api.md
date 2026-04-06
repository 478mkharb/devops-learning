# 🚀 Employee API — Production-Grade Setup Guide

![Go](https://img.shields.io/badge/Go-1.18+-00ADD8?logo=go)
![ScyllaDB](https://img.shields.io/badge/DB-ScyllaDB-green)
![License](https://img.shields.io/badge/License-MIT-blue)
![Status](https://img.shields.io/badge/Status-Working-brightgreen)

---

## 📌 Overview

`employee-api` is a **Go-based microservice** responsible for managing employee data using **ScyllaDB (Cassandra-compatible NoSQL database)**.

This README documents a **complete production-style setup (without Docker)** including:

* Native installation
* Infrastructure debugging
* Database provisioning
* API validation

---

## 🏗️ Architecture

```
Client → employee-api (Go + Gin) → ScyllaDB
```

* REST API built with Gin
* NoSQL datastore (ScyllaDB)
* Config-driven setup

---

## 🧰 Tech Stack

| Layer     | Technology       |
| --------- | ---------------- |
| Language  | Go               |
| Framework | Gin              |
| Database  | ScyllaDB         |
| Cache     | Redis (optional) |
| Config    | YAML             |

---

## ⚙️ Prerequisites

* Ubuntu 22.04
* sudo access
* Internet connectivity
* Minimum 1–2 GB RAM (developer mode required)

---

## 🪜 Installation Steps

### 1️⃣ Install Go

```bash
sudo apt update
sudo apt install golang-go -y
go version
```

---

### 2️⃣ Clone Repository

```bash
git clone <your-repo-url>
cd OT-Microservices/employee-api
```

---

### 3️⃣ Install Dependencies

```bash
go mod tidy
```

---

## 🗄️ ScyllaDB Setup (Official Method)

### 4️⃣ Install ScyllaDB

```bash
curl -sSf https://get.scylladb.com/server | sudo bash
sudo apt install scylla -y
```

---

### 5️⃣ Run Setup

```bash
sudo scylla_setup
```

Recommended choices:

* IOTune → YES
* Only service on host → NO

---

### 6️⃣ Enable Developer Mode (IMPORTANT)

```bash
sudo nano /etc/scylla/scylla.yaml
```

Update:

```yaml
developer_mode: true
```

Restart service:

```bash
sudo systemctl restart scylla-server
sudo systemctl status scylla-server
```

---

## 🧪 Database Setup

### 7️⃣ Connect to DB

```bash
cqlsh
```

---

### 8️⃣ Create Keyspace

```sql
CREATE KEYSPACE employee WITH replication = {
  'class': 'SimpleStrategy',
  'replication_factor': 1
};
```

---

### 9️⃣ Use Keyspace

```sql
USE employee;
```

---

### 🔟 Create Table

```sql
CREATE TABLE IF NOT EXISTS employee_info (
    id text,
    name text,
    designation text,
    department text,
    joining_date date,
    address text,
    office_location text,
    status text,
    email text,
    phone_number text,
    PRIMARY KEY (id, joining_date)
) WITH CLUSTERING ORDER BY (joining_date DESC);
```

---

## 🔧 Application Configuration

Edit `config.yaml`:

```yaml
scylladb:
  host: 127.0.0.1
  port: 9042
  keyspace: employee

redis:
  enabled: false
```

---

## ▶️ Run Application

```bash
go run main.go
```

Expected:

```
Listening and serving HTTP on :8080
```

---

## 🧪 API Testing

### 🔹 Health Check

```bash
curl http://localhost:8080/api/v1/employee/health
```

Expected:

```json
{"message":"Employee API is running"}
```

---

### 🔹 Create Employee

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

### 🔹 Fetch Employee

```bash
curl "http://localhost:8080/api/v1/employee/search?id=1"
```

---

## 🧪 Verify Data in DB

```bash
cqlsh
```

```sql
USE employee;
SELECT * FROM employee_info;
```

---

## ⚠️ Common Issues & Fixes

### ❌ Scylla not starting

```bash
journalctl -u scylla-server
```

✔ Fix: enable `developer_mode: true`

---

### ❌ Connection refused (8080)

✔ Ensure app is running
✔ Keep terminal open

---

### ❌ Keyspace not found

✔ Run:

```sql
USE employee;
```

---

### ❌ Table not found

✔ Run table creation query

---

## 🚀 Production Improvements (Next Steps)

* Run API via `systemd`
* Add NGINX reverse proxy
* Enable logging & monitoring (Prometheus)
* Add Redis caching
* Integrate with `attendance-api`

---

## 🧠 Key Learnings

* Handling broken package repositories
* Debugging systemd service failures
* Understanding Scylla IO constraints
* Using developer mode for low-resource environments
* Manual schema provisioning in NoSQL
* Service-to-database integration

---

## 📄 License

This project is licensed under the MIT License.

---

## 🙌 Acknowledgements

* ScyllaDB Documentation
* Gin Framework
* Open-source microservices architecture references

---

## ⭐ If this helped you

Give the repo a ⭐ and continue building 🚀
