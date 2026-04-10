# 🚀 OT-Microservices — Clean Manual Runbook (DEV MODE)

> Purpose: **Simple, reliable commands for demo** (Start → Verify → No mixing, No sleep required)

---

# 📁 Setup Logs

```bash
mkdir -p ~/logs
```

---

# 🧹 1. Cleanup (Ports)

```bash
fuser -k 8080/tcp
fuser -k 8081/tcp
fuser -k 8082/tcp
```

---

# 🧱 2. Start Infrastructure

```bash
sudo systemctl start scylla-server
sudo systemctl start postgresql
sudo systemctl start redis
sudo systemctl start nginx
```

---

# 🧠 3. Employee API

## ▶ Start

```bash
cd ~/OT-Micro/employee-api
nohup go run main.go > ~/logs/employee-api.log 2>&1 &
```

## ✅ Verify

```bash
lsof -i :8080
curl http://localhost:8080/api/v1/employee/health
```

---

# 🐍 4. Attendance API

## ▶ Start

```bash
cd ~/OT-Micro/attendance-api
nohup poetry run gunicorn app:app \
  --log-config log.conf \
  -b 0.0.0.0:8081 > ~/logs/attendance.log 2>&1 &
```

## ✅ Verify

```bash
lsof -i :8081
curl http://localhost:8081/api/v1/attendance/health
```

---

# ☕ 5. Salary API

## ▶ Start

```bash
cd ~/OT-Micro/salary-api
nohup ./mvnw spring-boot:run > ~/logs/salary.log 2>&1 &
```

## ✅ Verify

```bash
lsof -i :8082
curl http://localhost:8082/actuator/health
```

---

# 🔍 6. Swagger Validation

```bash
curl -I http://localhost:8080/swagger/index.html
curl -I http://localhost:8081/apidocs/
curl -I http://localhost:8082/swagger-ui/index.html
```

---

# 🌐 7. Access URLs

## UI

```
http://192.168.122.167/
```

## APIs

```
http://192.168.122.167:8080/api/v1/employee/health
http://192.168.122.167:8081/api/v1/attendance/health
http://192.168.122.167:8082/actuator/health
```

## Swagger

```
http://192.168.122.167:8080/swagger/index.html
http://192.168.122.167:8081/apidocs/
http://192.168.122.167:8082/swagger-ui/index.html
```

---

# 🗄️ 8. Database Demo Commands

## 🔹 ScyllaDB

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

## 🔹 PostgreSQL

```bash
psql -U postgres -h 127.0.0.1 -d attendance_db
```

```sql
\dt
SELECT * FROM attendance;
```

---

## 🔹 Redis

```bash
redis-cli
```

```bash
PING
KEYS *
```

---

# 📊 9. Logs

```bash
tail -f ~/logs/employee-api.log
tail -f ~/logs/attendance.log
tail -f ~/logs/salary.log
```

---

# 🧪 10. Quick API Test

```bash
curl http://localhost:8080/api/v1/employee/search/all | jq
curl http://localhost:8081/api/v1/attendance/search | jq
curl http://localhost:8082/api/v1/salary/search/all | jq
```

---

# 🧠 Demo Flow

1. Start Infra
2. Start APIs (one by one)
3. Verify APIs
4. Open UI
5. Show Swagger
6. Show DB data
7. Show logs

---

# ✅ Checklist

* [ ] Ports active (8080,8081,8082)
* [ ] APIs responding
* [ ] Swagger working
* [ ] UI accessible
* [ ] DB queries working

---

🔥 Clean • Simple • Demo Ready
