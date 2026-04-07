# Attendance API

---

## Objective

* Provision a fresh EC2 Ubuntu server
* Install all required dependencies
* Configure database and cache
* Setup Liquibase migrations
* Configure Poetry environment
* Run the application using Makefile

---

# Architecture Overview

```
Client → Flask App → Router Layer
                      ↓
              PostgreSQL (DB)
                      ↓
                  Redis (Cache)
```

---

# 📁 Directory Structure Explanation

## 🔹 Root Level

| Folder/File            | Purpose                              |
| ---------------------- | ------------------------------------ |
| `app.py`               | Entry point of Flask application     |
| `config.yaml`          | Stores DB & Redis configuration      |
| `pyproject.toml`       | Dependency management using Poetry   |
| `Makefile`             | Automation for build, run, migration |
| `liquibase.properties` | DB migration configuration           |
| `migration/`           | Contains Liquibase changelog         |
| `scripts/`             | Helper scripts (e.g., DB init)       |
| `Dockerfile`           | Container build (future use)         |
| `README.md`            | Documentation                        |

---

## 🔹 client/

Handles external systems (DB & Cache)

| Folder      | Purpose                     |
| ----------- | --------------------------- |
| `postgres/` | PostgreSQL connection logic |
| `redis/`    | Redis connection logic      |
| `tests/`    | Unit tests for client layer |

---

## 🔹 router/

| File            | Purpose        |
| --------------- | -------------- |
| `attendance.py` | API endpoints  |
| `cache.py`      | Cache handling |
| `tests/`        | API tests      |

---

## 🔹 models/

| File           | Purpose                   |
| -------------- | ------------------------- |
| `user_info.py` | Data model for attendance |
| `message.py`   | API response models       |
| `tests/`       | Model tests               |

---

## 🔹 utils/

| File              | Purpose                   |
| ----------------- | ------------------------- |
| `validator.py`    | Input validation          |
| `json_encoder.py` | Custom JSON serialization |
| `log_encoder.py`  | Structured logging        |
| `tests/`          | Utility tests             |

---

# STEP 1 — Launch EC2 Instance

* OS: Ubuntu 22.04
* Instance: t2.micro
* Open Ports:

  * 22 (SSH)
  * 8000 (App)

---

# STEP 2 — Connect to Server

```bash
ssh ubuntu@<EC2-IP>
```

---

# STEP 3 — Update System

```bash
sudo apt update && sudo apt upgrade -y
```

---

# STEP 4 — Install Base Packages

```bash
sudo apt install git curl wget unzip -y
```

---

# STEP 5 — Install Python 3.11

```bash
sudo apt install python3.11 python3.11-venv python3.11-distutils -y
```

Verify:

```bash
python3.11 --version
```

---

# STEP 6 — Install PostgreSQL

```bash
sudo apt install postgresql postgresql-contrib -y
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

### Setup DB

```bash
sudo -u postgres psql
```

```sql
CREATE DATABASE attendance_db;
ALTER USER postgres PASSWORD 'password';
```

Exit:

```bash
\q
```

---

# STEP 7 — Install Redis

```bash
sudo apt install redis-server -y
sudo systemctl start redis
sudo systemctl enable redis
```

Test:

```bash
redis-cli ping
```

---

# STEP 8 — Install Liquibase

```bash
wget https://github.com/liquibase/liquibase/releases/download/v4.27.0/liquibase-4.27.0.tar.gz

sudo mkdir -p /opt/liquibase

sudo tar -xvf liquibase-4.27.0.tar.gz -C /opt/liquibase --strip-components=1

sudo chmod +x /opt/liquibase/liquibase
sudo ln -s /opt/liquibase/liquibase /usr/local/bin/liquibase
```

Verify:

```bash
liquibase --version
```

---

# STEP 9 — Install Poetry

```bash
pip install --user poetry

echo 'export PATH=$PATH:$HOME/.local/bin' >> ~/.bashrc
source ~/.bashrc
```

---

# STEP 10 — Clone Project

```bash
git clone https://github.com/OT-MICROSERVICES/attendance-api.git
cd attendance-api
```

---

# STEP 11 — Fix pyproject.toml

### Remove:

```toml
packages = [{include = "attendance_api"}]
```

### Add:

```toml
package-mode = false
```

---

# STEP 12 — Configure Liquibase

```bash
nano liquibase.properties
```

```properties
changeLogFile=migration/db.changelog-master.xml

url=jdbc:postgresql://localhost:5432/attendance_db
username=postgres
password=password

driver=org.postgresql.Driver
classpath=/opt/liquibase/internal/lib/postgresql.jar
```

---

# STEP 13 — Update Makefile

```makefile
.PHONY: build fmt run test migrate setup

build:
	poetry install --no-root --no-interaction --no-ansi

fmt:
	poetry run pylint router/ client/ models/ utils/ app.py

run:
	poetry run python app.py

migrate:
	liquibase update --driver-properties-file=liquibase.properties

setup: build migrate run
```

---

# STEP 14 — Run Application

```bash
make setup
```

---

# STEP 15 — Test API

```bash
curl http://localhost:8000/api/v1/attendance/health
```

---

# Important Points

* Dependency isolation using Poetry
* DB version control using Liquibase
* Service automation using Makefile
* Layered microservice architecture

---


🔥 End of SOP
