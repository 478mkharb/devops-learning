# 🚀 OT-Microservices — Manual Runbook (DEV MODE)

> Purpose: **Single reference** to start all services manually, verify Swagger, and demonstrate databases during a live demo.

---

# 📁 Log Directory Setup

```bash
mkdir -p ~/logs
```

````

---

# 🧹 1. Cleanup (Ports)

```bash
fuser -k 8080/tcp
fuser -k 8081/tcp
fuser -k 8082/tcp
````

---

# 🧱 2. Start Infrastructure

```bash
sudo systemctl start scylla-server
sudo systemctl start postgresql
sudo systemctl start redis
sudo systemctl start nginx
```

## ✅ Verify Infra

```bash
sudo systemctl status scylla-server
sudo systemctl status postgresql
sudo systemctl status redis
sudo systemctl status nginx
```

---

# 🧠 3. Employee API (Go)

```bash
cd ~/OT-Micro/employee-api

# Run
nohup go run main.go > ~/logs/employee-api.log 2>&1 &

# Verify
lsof -i :8080
curl http://localhost:8080/api/v1/employee/health
```

---

# 🐍 4. Attendance API (Python + Gunicorn)

```bash
cd ~/OT-Micro/attendance-api

nohup poetry run gunicorn app:app \
  --log-config log.conf \
  -b 0.0.0.0:8081 > ~/logs/attendance.log 2>&1 &

# Verify
lsof -i :8081
curl http://localhost:8081/api/v1/attendance/health
```

---

# ☕ 5. Salary API (Spring Boot)

```bash
cd ~/OT-Micro/salary-api

nohup ./mvnw spring-boot:run > ~/logs/salary.log 2>&1 &

# Wait for startup
sleep 10

# Verify
lsof -i :8082
curl http://localhost:8082/actuator/health
```

---

# 🔍 6. Swagger Validation

```bash
# Employee
curl -I http://localhost:8080/swagger/index.html

# Attendance
curl -I http://localhost:8081/apidocs/

# Salary
curl -I http://localhost:8082/swagger-ui/index.html
```

---

# 🌐 7. Access URLs (Demo Ready)

## UI

```
http://192.168.122.167/
```

## APIs

```
Employee   → http://192.168.122.167:8080/api/v1/employee/health
Attendance → http://192.168.122.167:8081/api/v1/attendance/health
Salary     → http://192.168.122.167:8082/actuator/health
```

## Swagger

```
Employee   → http://192.168.122.167:8080/swagger/index.html
Attendance → http://192.168.122.167:8081/apidocs/
Salary     → http://192.168.122.167:8082/swagger-ui/index.html
```

---

# 🗄️ 8. Database Demo Commands (IMPORTANT FOR INTERVIEW)

## 🔹 ScyllaDB (Employee + Salary)

```bash
cqlsh
```

```sql
DESCRIBE KEYSPACES;
USE employee_db;
DESCRIBE TABLES;
SELECT * FROM employee;
```

### Salary DB

```sql
USE salary_keyspace;
DESCRIBE TABLES;
SELECT * FROM employee_salary;
```

---

## 🔹 PostgreSQL (Attendance)

```bash
psql -U postgres -h 127.0.0.1 -d attendance_db
```

```sql
\dt
SELECT * FROM attendance;
```

---

## 🔹 Redis (Cache Demo)

```bash
redis-cli
```

```bash
PING
KEYS *
GET "employee_*"
```

---

# 📊 9. Logs (Live Debug)

```bash
tail -f ~/logs/employee-api.log
tail -f ~/logs/attendance.log
tail -f ~/logs/salary.log
```

---

# 🧪 10. Quick API Test Commands

```bash
# Employee
curl http://localhost:8080/api/v1/employee/search/all | jq

# Attendance
curl http://localhost:8081/api/v1/attendance/search | jq

# Salary
curl http://localhost:8082/api/v1/salary/search/all | jq
```

---

# 🧠 Demo Flow (Use This in Interview)

1. Start infra
2. Start all APIs
3. Show UI (Nginx)
4. Open Swagger
5. Hit APIs via curl
6. Show DB data (cqlsh + psql)
7. Show logs

---

# ✅ Final Checklist

* [ ] All ports running (8080, 8081, 8082)
* [ ] Swagger accessible
* [ ] UI loads
* [ ] DB queries working
* [ ] Logs updating

---

🔥 Ready for Demo
