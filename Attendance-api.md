# 📘 Attendance API – Complete Setup & Learning Guide (EC2 Ubuntu 22 | No Docker)

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
Redis Cache
        ↓
Response
```

---

## 📁 Project Structure (Reference)

(From OT-Microservices repo)

* app.py → Flask entry point
* config.yaml → Config for DB & Redis
* router/attendance.py → API routes
* client/postgres → DB logic
* client/redis → Cache logic

---

# ⚙️ EC2 SETUP (Ubuntu 22.04)

---

## 🪜 STEP 1 — Launch EC2

* OS: Ubuntu 22.04
* Instance: t2.micro
* Open Ports:

  * 22 (SSH)
  * 8000 (App)

---

## 🪜 STEP 2 — Connect to EC2

```bash
ssh ubuntu@<EC2-PUBLIC-IP>
```

---

## 🪜 STEP 3 — System Update

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 🪜 STEP 4 — Install Base Packages

```bash
sudo apt install git curl wget unzip -y
```

---

## 🪜 STEP 5 — Install Python + Build Dependencies

```bash
sudo apt install python3.11 python3.11-venv python3-pip -y
sudo apt install libpq-dev gcc -y
```

👉 Required for psycopg2 build

---

## 🪜 STEP 6 — Install PostgreSQL

```bash
sudo apt install postgresql postgresql-contrib -y
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

---

### 🔹 Setup Database

```bash
sudo -u postgres psql
```

```sql
CREATE DATABASE attendance_db;
\c attendance_db

CREATE TABLE records (
    id VARCHAR(50),
    name VARCHAR(100),
    status VARCHAR(20),
    date DATE
);

ALTER USER postgres PASSWORD 'password';
```

```bash
\q
```

---

## 🪜 STEP 7 — Install Redis

```bash
sudo apt install redis-server -y
sudo systemctl start redis
sudo systemctl enable redis
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

## 🪜 STEP 8 — Clone Repository

```bash
git clone https://github.com/OT-MICROSERVICES/attendance-api.git
cd attendance-api
```

---

## 🪜 STEP 9 — Setup Python Virtual Environment

```bash
python3.11 -m venv venv
source venv/bin/activate
```

---

## 🪜 STEP 10 — Install Dependencies

### 🔹 Option A (Recommended)

```bash
pip install poetry
poetry install
```

---

### 🔹 Option B (Manual Fixes if errors come)

```bash
pip install flask flask-caching redis psycopg2-binary
```

---

## 🪜 STEP 11 — Fix Config (VERY IMPORTANT)

Edit file:

```bash
nano config.yaml
```

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

## 🪜 STEP 12 — Update app.py for local run

Add at bottom:

```python
if __name__ == "__main__":
    print("Starting Attendance API...")
    app.run(host="0.0.0.0", port=8000, debug=True)
```

---

## 🪜 STEP 13 — Run Application

```bash
python app.py
```

Expected:

```
Running on http://0.0.0.0:8000
```

---

## 🪜 STEP 14 — Test API (Inside EC2)

```bash
curl http://localhost:8000/api/v1/attendance/health
```

---

## 🪜 STEP 15 — Test from Browser

```text
http://<EC2-PUBLIC-IP>:8000/api/v1/attendance/health
```

---

# 🧪 API Testing

### 🔹 Create Record

```bash
curl -X POST http://<EC2-IP>:8000/api/v1/attendance/create \
-H "Content-Type: application/json" \
-d '{"id":"1","name":"Mukesh","status":"present","date":"2026-04-03"}'
```

---

### 🔹 Fetch Record

```bash
curl "http://<EC2-IP>:8000/api/v1/attendance/search?id=1"
```

---

# ⚡ Redis Behavior

* First request → DB hit
* Next request → Redis cache

---

# 🧨 Failure Testing

### 🔴 Stop Redis

```bash
sudo systemctl stop redis
```

👉 API fails (tight coupling)

---

### 🔴 Start Redis

```bash
sudo systemctl start redis
```

---

### 🔴 Stop PostgreSQL

```bash
sudo systemctl stop postgresql
```

👉 API fails completely

---

# 🧠 Key Learnings

### 🔹 Infra Setup

* EC2 provisioning
* Package installation
* Service management (systemctl)

---

### 🔹 Backend Concepts

* Flask API flow
* Routing
* DB interaction
* Cache layer

---

### 🔹 Debugging

* psycopg2 missing
* Redis connection refused
* DB schema missing

---

# 🚀 Production Improvements

* Gunicorn (WSGI server)
* NGINX reverse proxy
* systemd service for app
* Separate DB instance

---

# 🧠 Summary

```
EC2 + Flask + PostgreSQL + Redis + Debugging
```

---

🔥 End of EC2 Guide
