# 📘 Attendance API — End-to-End SOP (Ubuntu 22 EC2)

---

## 🎯 Objective

This document provides a **step-by-step, production-oriented setup** of the Attendance Microservice from scratch on a fresh Ubuntu 22 EC2 instance.

It covers:

* System preparation
* Dependency installation
* Database + cache setup
* Migration using Liquibase
* Application execution (Flask + Gunicorn)
* Validation & troubleshooting

---

# 🧱 Architecture Overview

```
Client → Gunicorn → Flask App → Router Layer
                                  ↓
                         PostgreSQL (DB)
                                  ↓
                               Redis (Cache)
```

---

# 📁 Directory Structure

```
attendance-api/
├── app.py                # Entry point (Flask app instance)
├── config.yaml           # DB + Redis configuration
├── pyproject.toml        # Dependency management (Poetry)
├── Makefile              # Automation (optional usage)
├── liquibase.properties  # DB migration config
├── migration/            # Liquibase changelog
├── client/               # DB & Redis connectors
├── router/               # API routes
├── models/               # Data models
├── utils/                # Helpers (validation, logging)
```

---

# 🚀 STEP 1 — System Preparation

## Why?

Ensure system is updated and basic tools are available.

**Detailed Explanation:**

* `apt update` refreshes package index so latest versions can be installed
* `apt upgrade` ensures system packages are patched (security + compatibility)
* Tools installed:

  * `git` → required to clone repository
  * `curl/wget` → used for downloading binaries (Liquibase, Go, etc.)
  * `unzip` → used for extracting archives
* Skipping this step may lead to dependency mismatch or installation failures

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git curl wget unzip
```

---

# 🐍 STEP 2 — Install Python 3.11

## Why?

Project requires modern Python runtime compatible with dependencies.

**Detailed Explanation:**

* The project dependencies (Flask, Peewee, Redis client) are built/tested on Python 3.11
* Older versions may cause runtime or installation issues
* `python3.11-venv` → enables isolated virtual environments
* `python3.11-distutils` → required by some build tools
* Ensures consistency across environments (local, EC2, CI/CD)

```bash
sudo apt install -y python3.11 python3.11-venv python3.11-distutils
python3.11 --version
```

---

# 🗄️ STEP 3 — Install PostgreSQL

## Why?

Primary database to store attendance records.

**Detailed Explanation:**

* PostgreSQL is a reliable relational database used for structured data
* Stores attendance records in normalized schema
* ACID compliance ensures data integrity
* Used by Peewee ORM in this project
* `systemctl enable` ensures DB starts automatically on reboot

**DB Setup Purpose:**

* `attendance_db` → dedicated database for this service
* Setting password ensures authentication for application access

```bash
sudo apt install -y postgresql postgresql-contrib
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

## Setup DB

```bash
sudo -u postgres psql
```

```sql
CREATE DATABASE attendance_db;
ALTER USER postgres PASSWORD 'password';
```

---

# ⚡ STEP 4 — Install Redis

## Why?

Used for caching layer to improve performance.

**Detailed Explanation:**

* Redis is an in-memory datastore → extremely fast reads/writes
* Used to cache frequently accessed data
* Reduces load on PostgreSQL
* Improves API response time
* Helps in scalability (high request handling)
* `redis-cli ping` ensures connectivity

**In this project:**

* Redis is used via client layer (`client/redis`)
* Optional but recommended for performance optimization

```bash
sudo apt install -y redis-server
sudo systemctl start redis
sudo systemctl enable redis
redis-cli ping
```

Expected: `PONG`

---

# 🔄 STEP 5 — Install Liquibase

## Why?

Liquibase manages **database schema versioning** using changelog.

**Detailed Explanation:**

* Acts like Git for database schema
* Uses XML changelog (`db.changelog-master.xml`) as source of truth
* Automatically applies schema changes (tables, indexes, constraints)
* Tracks applied changes in `databasechangelog` table
* Prevents duplicate execution of migrations

**Benefits:**

* Consistent DB structure across environments
* No manual SQL execution required
* Safe incremental updates
* Easy rollback support

**Why critical here:**

* Project does NOT include raw SQL setup scripts
* Without Liquibase, tables won’t exist → API fails

```bash
wget https://github.com/liquibase/liquibase/releases/download/v4.27.0/liquibase-4.27.0.tar.gz
sudo mkdir -p /opt/liquibase
sudo tar -xvf liquibase-4.27.0.tar.gz -C /opt/liquibase
sudo chmod +x /opt/liquibase/liquibase
sudo ln -s /opt/liquibase/liquibase /usr/local/bin/liquibase
```

Verify:

```bash
liquibase --version
```

---

# 📦 STEP 6 — Install Poetry

## Why?

Poetry manages dependencies and virtual environments cleanly.

**Detailed Explanation:**

* Replaces pip + venv with unified tool
* Handles dependency resolution via `pyproject.toml`
* Creates isolated virtual environment
* Ensures same dependency versions using `poetry.lock`

**Benefits:**

* No global package conflicts
* Reproducible builds
* Cleaner dependency management

**In this project:**

* All libraries (Flask, Redis, Peewee) are defined in Poetry config
* Must use `poetry run` to execute application

```bash
pip install --user poetry

echo 'export PATH=$PATH:$HOME/.local/bin' >> ~/.bashrc
source ~/.bashrc

poetry --version
```

---

# 📥 STEP 7 — Clone Project

```bash
git clone https://github.com/OT-MICROSERVICES/attendance-api.git
cd attendance-api
```

---

# ⚙️ STEP 8 — Fix pyproject.toml

## Why?

Prevent Poetry packaging issues.

**Detailed Explanation:**

* Project is not structured as a Python package (no proper module layout)
* Default Poetry behavior expects installable package
* Removing `packages` avoids packaging errors
* Adding `package-mode = false` tells Poetry:
  → "Just manage dependencies, don’t treat as package"

**Without this fix:**

* `poetry install` may fail or behave incorrectly

Remove:

```
packages = [{include = "attendance_api"}]
```

Add:

```
package-mode = false
```

---

# 📦 STEP 9 — Install Dependencies

```bash
poetry install --no-root --no-interaction --no-ansi
```

---

# 🔗 STEP 10 — Configure Liquibase

Edit:

```bash
nano liquibase.properties
```

Set:

```properties
url=jdbc:postgresql://127.0.0.1:5432/attendance_db
driver=org.postgresql.Driver
username=postgres
password=password
changeLogFile=migration/db.changelog-master.xml
classpath=/opt/liquibase/internal/lib/postgresql.jar
```

---

# 🧬 STEP 11 — Run DB Migration

## Why?

Creates required tables automatically.

**Detailed Explanation:**

* Reads changelog XML file
* Executes SQL operations defined in changesets
* Creates:

  * Application tables (e.g., `records`)
  * Liquibase tracking tables

**Verification importance:**

* Ensures DB schema exists before app starts
* Missing tables = runtime errors in API

**Internal working:**

* Liquibase checks `databasechangelog`
* Applies only new changes

```bash
liquibase update --defaultsFile=liquibase.properties
```

Verify:

```bash
sudo -u postgres psql
\c attendance_db
\dt
```

---

# 🧠 STEP 12 — Application Config

Edit:

```bash
nano config.yaml
```

Ensure:

```yaml
postgres:
  database: attendance_db
  host: 127.0.0.1
  port: 5432
  user: postgres
  password: password

redis:
  host: 127.0.0.1
  port: 6379
  password: ""
```

---

# ▶️ STEP 13 — Run Application (Development)

## Why?

Flask CLI is required to properly start app.

**Detailed Explanation:**

* `app.py` defines Flask app but may not call `app.run()` explicitly
* Flask CLI loads app context correctly
* Enables environment-based execution

**Why not `python app.py`:**

* May not trigger server start
* Silent failures possible

**Flask CLI advantages:**

* Better debugging
* Proper app discovery
* Supports environment configs

```bash
poetry run flask --app app run --host=0.0.0.0 --port=8000
```

---

# 🔎 STEP 14 — Verify API

```bash
curl http://localhost:8000/api/v1/attendance/health
```

Expected:

```json
{"message":"Attendance API is running fine and ready to serve requests"}
```

---

# 📘 STEP 15 — Swagger UI

Access:

```
http://<SERVER-IP>:8000/apidocs/
```

---

# 🚀 STEP 16 — Run with Gunicorn (Production)

## Why?

Gunicorn provides:

* Multi-worker support
* Stability
* Production readiness

**Detailed Explanation:**

* Gunicorn is a WSGI HTTP server for Python apps
* Sits between client and Flask app

**How it works:**
Client → Gunicorn → Flask

**Key advantages:**

* Multiple workers handle concurrent requests
* Better CPU utilization
* Automatic worker restart on failure
* Production-safe compared to Flask dev server

**Why needed in production:**

* Flask dev server is single-threaded and not secure
* Gunicorn ensures high availability and performance

**Worker concept:**

* `-w 2` → 2 parallel processes handling requests
* More workers = better concurrency (within limits)

```bash
poetry add gunicorn
poetry run gunicorn -w 2 -b 0.0.0.0:8000 app:app
```

---

# 🔧 STEP 17 — Configure systemd Service (Gunicorn Auto-Start)

## Why?

Ensure the API runs as a **background service**, starts on boot, and auto-restarts on failure.

**Detailed Explanation:**

* `systemd` manages Linux services (start/stop/restart/status)
* Keeps Gunicorn running independently of terminal session
* Restarts service automatically if it crashes
* Enables auto-start on system reboot
* Standard practice for production deployments

---

## 🔧 Create Service File

```bash
sudo nano /etc/systemd/system/attendance.service
```

---

## 📄 Add Configuration

```ini
[Unit]
Description=Attendance API Gunicorn Service
After=network.target

[Service]
User=mukesh
WorkingDirectory=/home/mukesh/attendance-api
ExecStart=/home/mukesh/.local/bin/poetry run gunicorn -w 2 -b 0.0.0.0:8000 app:app
Restart=always

[Install]
WantedBy=multi-user.target
```

---

## ⚠️ Important Notes

* Replace `mukesh` with your EC2 username if different
* Ensure correct project path
* Ensure Poetry path is correct:

  ```bash
  which poetry
  ```

---

## 🔄 Reload systemd

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
```

---

## ▶️ Start Service

```bash
sudo systemctl start attendance
```

---

## 🔁 Enable Auto Start

```bash
sudo systemctl enable attendance
```

---

## 🔍 Verify

```bash
sudo systemctl status attendance
```

Expected:

```
Active: active (running)
```

---

## 🔎 Test API

```bash
curl http://localhost:8000/api/v1/attendance/health
```

---

## 🧠 Service Lifecycle Commands

```bash
sudo systemctl stop attendance
sudo systemctl start attendance
sudo systemctl restart attendance
sudo systemctl status attendance
```

---

# ✅ Final Validation Checklist

* [x] PostgreSQL running
* [x] Redis running
* [x] Liquibase migration success
* [x] API responding
* [x] Swagger accessible
* [x] Gunicorn running

---

# 🛠️ Troubleshooting

## App not starting

* Use Flask CLI instead of `python app.py`

## DB connection error

* Check config.yaml credentials

## Migration failure

* Verify liquibase.properties

## Port not open

```bash
lsof -i :8000
```

---

# 🏁 Conclusion

This SOP ensures:

* Reproducible setup
* Clean dependency management
* Production-grade runtime
* Proper microservice architecture adherence

---

🔥 Attendance API is now fully operational and production-ready.
