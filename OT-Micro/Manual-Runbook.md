# OT-Microservices — Manual Runbook

<p align="center">
  <img src="https://img.shields.io/badge/Go-EmployeeAPI-blue">
  <img src="https://img.shields.io/badge/Python-AttendanceAPI-green">
  <img src="https://img.shields.io/badge/Java-SalaryAPI-orange">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Flask-NotificationAPI-lightgrey">
  <img src="https://img.shields.io/badge/ScyllaDB-Database-blueviolet">
  <img src="https://img.shields.io/badge/PostgreSQL-AttendanceDB-blue">
  <img src="https://img.shields.io/badge/NGINX-ReverseProxy-red">
</p>

---

## Setup Logs

```bash
mkdir -p ~/logs
```

---

## Initial Cleanup (Ports)

```bash
fuser -k 8080/tcp 2>/dev/null
fuser -k 8081/tcp 2>/dev/null
fuser -k 8082/tcp 2>/dev/null
fuser -k 5000/tcp 2>/dev/null
```

---

## Start Infrastructure

```bash
sudo systemctl start scylla-server
sudo systemctl start postgresql
sudo systemctl start redis
sudo systemctl start nginx
```

---

## Employee API (Go)

### Start

```bash
cd ~/OT-Micro/employee-api
nohup go run main.go > ~/logs/employee-api.log 2>&1 &
```

### Verify

```bash
lsof -i :8080
curl http://localhost:8080/api/v1/employee/health
```

---

## Attendance API (Python + PostgreSQL)

### Start

```bash
cd ~/OT-Micro/attendance-api
nohup poetry run gunicorn app:app \
  --log-config log.conf \
  -b 0.0.0.0:8081 > ~/logs/attendance.log 2>&1 &
```

### Verify

```bash
lsof -i :8081
curl http://localhost:8081/api/v1/attendance/health
```

---

## Salary API (Java)

### Start

```bash
cd ~/OT-Micro/salary-api
nohup ./mvnw spring-boot:run > ~/logs/salary.log 2>&1 &
```

### Verify

```bash
lsof -i :8082
curl http://localhost:8082/actuator/health
```

---

## Notification API (Python)

### Start

```bash
cd ~/OT-Micro/notification-worker
pip3 install -r requirements.txt
nohup python3 notification_api.py --mode api > ~/logs/notification.log 2>&1 &
```

### Verify

```bash
lsof -i :5000
curl http://localhost:5000/health
```

---

## Swagger Validation

```bash
curl -I http://localhost:8080/swagger/index.html
curl -I http://localhost:8081/apidocs/
curl -I http://localhost:8082/swagger-ui/index.html
```

---

## Access URLs

### UI

```
http://192.168.122.167/
```

### APIs

```
http://192.168.122.167:8080/api/v1/employee/health
http://192.168.122.167:8081/api/v1/attendance/health
http://192.168.122.167:8082/actuator/health
http://192.168.122.167:5000/health
```

### Swagger

```
http://192.168.122.167:8080/swagger/index.html
http://192.168.122.167:8081/apidocs/
http://192.168.122.167:8082/swagger-ui/index.html
```

---

## Database Commands

### ScyllaDB

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

---

### PostgreSQL

```bash
psql -U postgres -h 127.0.0.1 -d attendance_db
```

```sql
\dt
SELECT * FROM attendance;
```

---

### Redis

```bash
redis-cli
```

```bash
PING
KEYS *
```

---

## Logs

```bash
tail -f ~/logs/employee-api.log
tail -f ~/logs/attendance.log
tail -f ~/logs/salary.log
tail -f ~/logs/notification.log
```

---

## Functional API Tests

```bash
curl http://localhost:8080/api/v1/employee/search/all | jq
curl http://localhost:8081/api/v1/attendance/search | jq
curl http://localhost:8082/api/v1/salary/search/all | jq
```

---

## Notification Test

```bash
curl -X POST http://localhost:5000/notification/send \
-H "Content-Type: application/json" \
-d '{"id":"1","name":"Mukesh","email":"your-email@gmail.com"}'
```

Expected:

* PDF download
* Email received

---

## Stop All Services

```bash
fuser -k 8080/tcp
fuser -k 8081/tcp
fuser -k 8082/tcp
fuser -k 5000/tcp
```
---

## Repository

[https://github.com/opstree/OT-Microservices](https://github.com/opstree/OT-Microservices)

---
