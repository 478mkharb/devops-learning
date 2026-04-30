# OT-Microservices Fresh EC2 Deployment Runbook (Updated)

## Objective

Deploy the OT-Microservices project on a fresh Ubuntu 22.04 EC2 instance using systemd services, NGINX reverse proxy, Redis, PostgreSQL, ScyllaDB, Elasticsearch, and all application services.

---

## Architecture (Actual Running State)

* Frontend: React served by NGINX
* Employee API: Go + Redis + ScyllaDB
* Attendance API: Python Flask/Gunicorn + Redis + PostgreSQL + Elasticsearch index support
* Salary API: Spring Boot + Redis + ScyllaDB + Elasticsearch sync
* Notification API: Python + Gunicorn + PDF + SMTP Email
* Reverse Proxy: NGINX

---

## 1. Base Server Prep

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git curl unzip nginx redis-server postgresql python3-pip python3-venv openjdk-17-jdk maven nodejs npm
```

Enable core services:

```bash
sudo systemctl enable nginx redis-server postgresql
sudo systemctl start nginx redis-server postgresql
```

---

## 2. Clone Repository

```bash
cd ~
git clone https://github.com/OT-MICROSERVICES/OT-Microservices.git OT-Micro
cd ~/OT-Micro
```

---

## 3. Databases

## PostgreSQL (Attendance)

```bash
sudo -u postgres psql
CREATE DATABASE attendance_db;
\q
```

## ScyllaDB / CQL (Employee + Salary)

Install ScyllaDB if not already present, then ensure port 9042 is active.

```bash
systemctl status scylla-server
```

Keyspaces commonly used:

* employee_db
* salary_keyspace

---

## 4. Redis Verification

```bash
redis-cli ping
# Expected: PONG
```

---

## 5. Elasticsearch

Install / verify:

```bash
systemctl status elasticsearch
curl localhost:9200
```

Expected indexes later:

* attendance_records
* salary_records

---

## 6. Employee API (Go)

```bash
cd ~/OT-Micro/employee-api
go build -o employee-api .
```

Ensure config uses Redis host with port:

```yaml
redis:
  host: "127.0.0.1:6379"
```

Systemd:

```bash
sudo nano /etc/systemd/system/employee-api.service
```

```ini
[Unit]
Description=OT Micro Employee API
After=network.target redis-server.service

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/OT-Micro/employee-api
ExecStart=/home/ubuntu/OT-Micro/employee-api/employee-api
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now employee-api
```

---

## 7. Attendance API (Python)

```bash
cd ~/OT-Micro/attendance-api
pip3 install poetry
poetry install
```

Run via Gunicorn:

```ini
[Unit]
Description=OT Micro Attendance API
After=network.target redis-server.service postgresql.service

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/OT-Micro/attendance-api
ExecStart=/home/ubuntu/.local/bin/poetry run gunicorn app:app -b 0.0.0.0:8081
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl enable --now attendance-api
```

---

## 8. Salary API (Spring Boot)

```bash
cd ~/OT-Micro/salary-api
mvn clean package -DskipTests
```

Systemd:

```ini
[Unit]
Description=OT Micro Salary API
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/OT-Micro/salary-api
ExecStart=/usr/bin/java -jar /home/ubuntu/OT-Micro/salary-api/target/salary-0.1.0-RELEASE.jar
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl enable --now salary-api
```

Notes:

* Notification endpoint should use `http://localhost/notification/send`
* Add HTTP timeouts in Java outbound calls
* Salary syncs to Elasticsearch `salary_records`

---

## 9. Notification API (Gunicorn)

```bash
cd ~/OT-Micro/notification-worker
pip3 install flask gunicorn requests reportlab
```

Systemd:

```ini
[Unit]
Description=OT Micro Notification API
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/OT-Micro/notification-worker
ExecStart=/home/ubuntu/.local/bin/gunicorn -w 2 -b 0.0.0.0:5000 notification_api:app
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl enable --now notification-api
```

---

## 10. Frontend Build

```bash
cd ~/OT-Micro/frontend
npm install
npm run build
sudo rm -rf /var/www/html/*
sudo cp -r build/* /var/www/html/
```

---

## 11. NGINX Reverse Proxy

```nginx
server {
    listen 80;
    root /var/www/html;
    index index.html;

    location / {
        try_files $uri /index.html;
    }

    location /api/v1/employee/ {
        proxy_pass http://127.0.0.1:8080;
    }

    location /api/v1/attendance/ {
        proxy_pass http://127.0.0.1:8081;
    }

    location /api/v1/salary/ {
        proxy_pass http://127.0.0.1:8082;
    }

    location /notification/ {
        proxy_pass http://127.0.0.1:5000/notification/;
    }
}
```

```bash
sudo nginx -t && sudo systemctl restart nginx
```

---

## 12. Validation Checklist

```bash
curl http://localhost/api/v1/employee/health
curl http://localhost/api/v1/attendance/health
curl http://localhost:8082/actuator/health
curl http://localhost:5000/health
curl localhost:9200/_cat/indices?v
redis-cli keys '*'
```

---
