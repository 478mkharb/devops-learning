# 📘 Attendance API – Complete Setup & Learning Guide (No Docker)

---

## 🧠 Project Overview

This is a Python-based microservice built using **Flask**, designed to:

* Create attendance records
* Fetch attendance records
* Perform health checks
* Use **PostgreSQL** as database
* Use **Redis** as cache layer

---

## 🏗️ Architecture Flow

```
Client (curl/browser)
        ↓
Flask App (app.py)
        ↓
Router (attendance.py)
        ↓
Database Layer (PostgreSQL)
        ↓
(Optional) Redis Cache
        ↓
Response
```

---

## 📁 Directory Structure (Important Files)

| File/Folder          | Purpose                            |
| -------------------- | ---------------------------------- |
| app.py               | Entry point, initializes Flask app |
| config.yaml          | Stores DB & Redis configuration    |
| router/attendance.py | Contains API endpoints             |
| client/postgres      | DB connection logic                |
| client/redis         | Redis connection logic             |
| models               | Data structures                    |

---

## ⚙️ Prerequisites (Ubuntu 22 VM)

Install required tools:

```bash
sudo apt update
sudo apt install python3.11 python3.11-venv python3-pip git postgresql redis-server -y
```

---

## 🐍 Python Environment Setup

```bash
cd attendance-api
python3.11 -m venv venv
source venv/bin/activate
pip install poetry
poetry install
```

---

## 🗄️ PostgreSQL Setup

Start service:

```bash
sudo systemctl start postgresql
```

Open PostgreSQL:

```bash
sudo -u postgres psql
```

Create DB:

```sql
CREATE DATABASE attendance_db;
```

Switch DB:

```sql
\c attendance_db
```

Create table:

```sql
CREATE TABLE records (
    id VARCHAR(50),
    name VARCHAR(100),
    status VARCHAR(20),
    date DATE
);
```

Set password:

```sql
ALTER USER postgres PASSWORD 'password';
```

Exit:

```sql
\q
```

---

## 🔴 Redis Setup

Start Redis:

```bash
sudo systemctl start redis
```

Test:

```bash
redis-cli ping
```

Expected:

```
PONG
```

---

## ⚙️ Config File Fix (CRITICAL)

Edit `config.yaml`:

```yaml
postgres:
  database: attendance_db
  host: localhost
  port: 5432
  user: postgres
  password: password

redis:
  host: localhost
  port: 6379
  password: ""
```

---

## 🚀 Running the Application

Add this in `app.py` (if not present):

```python
if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8000, debug=True)
```

Run:

```bash
python app.py
```

Expected:

```
Running on http://127.0.0.1:8000
```

---

## 🧪 API Testing

### 🔹 Health Check

```bash
curl http://localhost:8000/api/v1/attendance/health
```

---

### 🔹 Create Record

```bash
curl -X POST http://localhost:8000/api/v1/attendance/create \
-H "Content-Type: application/json" \
-d '{"id":"1","name":"Mukesh","status":"present","date":"2026-04-03"}'
```

---

### 🔹 Fetch Record

```bash
curl "http://localhost:8000/api/v1/attendance/search?id=1"
```

---

## ⚡ Redis Caching Behavior

* First request → DB hit
* Next requests → Redis cache

Check Redis:

```bash
redis-cli
KEYS *
```

---

## 🧨 Failure Testing (VERY IMPORTANT)

### 🔹 Stop Redis

```bash
sudo systemctl stop redis
```

👉 API fails (current implementation)

---

### 🔹 Start Redis

```bash
sudo systemctl start redis
```

👉 API works again

---

### 🔹 Stop PostgreSQL

```bash
sudo systemctl stop postgresql
```

👉 API completely fails

---

## 🧠 Key Learnings

### 🔹 1. Dependency Types

| Component  | Type                                       |
| ---------- | ------------------------------------------ |
| PostgreSQL | Hard dependency                            |
| Redis      | Soft dependency (but tightly coupled here) |

---

### 🔹 2. Common Errors Faced

* Python version mismatch
* Missing dependencies (Flask, psycopg2)
* Wrong DB host (Docker vs localhost)
* Table not created
* Redis not running

---

### 🔹 3. Important Concepts

* Flask routing
* REST APIs
* Database schema
* Redis caching
* Debugging services

---

## 🚀 Next Improvements (Production Level)

* Use Gunicorn instead of Flask dev server
* Add NGINX reverse proxy
* Implement Redis fallback (graceful handling)
* Add logging & monitoring
* Create systemd service

---

## 🧠 Summary

This project helped understand:

```
Backend + Database + Cache + Debugging + Infra
```

---

## 👨‍💻 Author Notes

This setup was done manually on VM (no Docker) to understand:

* Real service dependencies
* System-level debugging
* DevOps workflow

---

🔥 End of Guide
