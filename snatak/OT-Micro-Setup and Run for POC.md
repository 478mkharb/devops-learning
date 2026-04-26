<div align="center">

<h1>OT-Microservices</h1>
<h3>Setup and Manual Runbook for POC</h3>

<br/>

<p>
  <a href="https://go.dev/">
    <img src="https://img.shields.io/badge/Go-EmployeeAPI-blue?style=for-the-badge" />
  </a>
  <a href="https://www.python.org/">
    <img src="https://img.shields.io/badge/Python-AttendanceAPI-green?style=for-the-badge" />
  </a>
  <a href="https://www.java.com/">
    <img src="https://img.shields.io/badge/Java-SalaryAPI-orange?style=for-the-badge" />
  </a>
</p>

<p>
  <a href="https://flask.palletsprojects.com/">
    <img src="https://img.shields.io/badge/Flask-NotificationAPI-lightgrey?style=for-the-badge" />
  </a>
  <a href="https://www.scylladb.com/">
    <img src="https://img.shields.io/badge/ScyllaDB-Database-blueviolet?style=for-the-badge" />
  </a>
  <a href="https://www.postgresql.org/">
    <img src="https://img.shields.io/badge/PostgreSQL-AttendanceDB-blue?style=for-the-badge" />
  </a>
  <a href="https://nginx.org/">
    <img src="https://img.shields.io/badge/NGINX-ReverseProxy-red?style=for-the-badge" />
  </a>
</p>

</div>

---

| Author       | Created on | Version | Last updated by | Last edited on | Pre Reviewer | L0 Reviewer | L1 Reviewer | L2 Reviewer  |
| ------------ | ---------- | ------- | --------------- | -------------- | ------------ | ----------- | ----------- | ------------ |
| Mukesh Kharb | 25/04/2026 | 1.0     | Mukesh Kharb    | 25/04/2026     | Team         | Mohit Kumar | Faisal Khan | Mahesh Kumar |

---

## Table of Contents

* [Introduction](#introduction)
* [Execution Steps](#execution-steps)
* [Verification](#verification)
* [Stop Services](#stop-services)
* [FAQs](#faqs)
* [Repository](#repository)
* [Contact Information](#contact-information)
* [References](#references)

---

## Introduction

This document provides a structured runbook to set up and execute the OT-Microservices stack for POC. It ensures proper sequencing of infrastructure, services, validation, and shutdown.

---

## Setup for Execution

### Setup Logs

```bash
mkdir -p ~/logs
```

### Initial Cleanup (Ports)

```bash
fuser -k 8080/tcp 2>/dev/null
fuser -k 8081/tcp 2>/dev/null
fuser -k 8082/tcp 2>/dev/null
fuser -k 5000/tcp 2>/dev/null
```

### Start Infrastructure

```bash
sudo systemctl start scylla-server
sudo systemctl start postgresql
sudo systemctl start redis
sudo systemctl start nginx
```

### Employee API (Go)

```bash
cd ~/OT-Micro/employee-api
nohup go run main.go > ~/logs/employee-api.log 2>&1 &
```

### Attendance API (Python)

```bash
cd ~/OT-Micro/attendance-api
nohup poetry run gunicorn app:app \
  --log-config log.conf \
  -b 0.0.0.0:8081 > ~/logs/attendance.log 2>&1 &
```

### Salary API (Java)

```bash
cd ~/OT-Micro/salary-api
nohup ./mvnw spring-boot:run > ~/logs/salary.log 2>&1 &
```

### Notification API (Python)

```bash
cd ~/OT-Micro/notification-worker
pip3 install -r requirements.txt
nohup python3 notification_api.py --mode api > ~/logs/notification.log 2>&1 &
```

### Database Commands

#### ScyllaDB

```bash
cqlsh
```

```sql
DESCRIBE KEYSPACES;
USE employee_db;
SELECT * FROM employee;
USE salary_keyspace;
SELECT * FROM employee_salary;
```

#### PostgreSQL

```bash
psql -U postgres -h 127.0.0.1 -d attendance_db
```

```sql
\dt
SELECT * FROM attendance;
```

#### Redis

```bash
redis-cli
```

```bash
PING
KEYS *
```

---

## Verification

### Functional API Tests

```bash
curl http://localhost:8080/api/v1/employee/search/all | jq
curl http://localhost:8081/api/v1/attendance/search | jq
curl http://localhost:8082/api/v1/salary/search/all | jq
```

### Swagger Validation

```bash
curl -I http://localhost:8080/swagger/index.html
curl -I http://localhost:8081/apidocs/
curl -I http://localhost:8082/swagger-ui/index.html
```

### Logs

```bash
tail -f ~/logs/employee-api.log
tail -f ~/logs/attendance.log
tail -f ~/logs/salary.log
tail -f ~/logs/notification.log
```

### Notification Test

```bash
curl -X POST http://localhost:5000/notification/send \
-H "Content-Type: application/json" \
-d '{"id":"1","name":"Mukesh","email":"your-email@gmail.com"}'
```

---

## Stop Services

```bash
fuser -k 8080/tcp
fuser -k 8081/tcp
fuser -k 8082/tcp
fuser -k 5000/tcp
sudo systemctl stop nginx
```

---

## FAQs

**Why are services not starting?**

> Ports may be occupied. Run cleanup before starting.

**Why APIs are not accessible?**

> Ensure services and infrastructure are running.

**Why logs are empty?**

> Verify nohup execution and log directory.

---

## Repository

[https://github.com/opstree/OT-Microservices](https://github.com/opstree/OT-Microservices)

---

## Contact Information

| Name         | Email                                                                             |
| ------------ | --------------------------------------------------------------------------------- |
| Mukesh Kharb | [mukesh.Kharb.snaatak@mygurukulam.co](mailto:mukesh.Kharb.snaatak@mygurukulam.co) |

---

## References

| Resource        | Link                                                                 |
| --------------- | -------------------------------------------------------------------- |
| Git Docs        | [https://git-scm.com/docs](https://git-scm.com/docs)                 |
| NGINX Docs      | [https://nginx.org/en/docs/](https://nginx.org/en/docs/)             |
| PostgreSQL Docs | [https://www.postgresql.org/docs/](https://www.postgresql.org/docs/) |
| ScyllaDB Docs   | [https://docs.scylladb.com/](https://docs.scylladb.com/)             |

---
