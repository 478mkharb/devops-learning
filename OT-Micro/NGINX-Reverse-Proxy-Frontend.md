# 📘 NGINX Reverse Proxy Setup for Frontend (OT Microservices)

This document provides a **complete step-by-step SOP (Standard Operating Procedure)** to configure **NGINX as a reverse proxy** for the React frontend and backend microservices.

---

# 🧩 Architecture Overview

```text
Browser (Client)
       ↓
     NGINX (Port 80)
       ↓
----------------------------------
|  /api/v1/employee   → 8080     |
|  /api/v1/attendance → 8000     |
|  /api/v1/salary     → 8082     |
----------------------------------
       ↓
 React Build (Static Files)
```

---

# ⚙️ Prerequisites

| Component    | Requirement             |
| ------------ | ----------------------- |
| OS           | Ubuntu 20.04 / 22.04    |
| NGINX        | Installed               |
| Frontend     | Built (`npm run build`) |
| Backend APIs | Running                 |

---

# 🔍 Step 1 — Install NGINX

```bash
sudo apt update
sudo apt install nginx -y
```

---

# 🔍 Step 2 — Verify NGINX

```bash
systemctl status nginx
```

✔ Expected: `active (running)`

---

# 📁 Step 3 — Frontend Build Setup

Ensure build exists:

```bash
ls ~/frontend/build
```

✔ Must contain:

* index.html
* static/

---

# ⚙️ Step 4 — Create NGINX Config

```bash
sudo nano /etc/nginx/sites-available/frontend
```

---

# 🔥 FINAL CONFIGURATION

```nginx
server {
    listen 80;
    server_name _;

    root /home/mukesh/OT-Micro/frontend/build;
    index index.html;

    # =========================
    # EMPLOYEE API
    # =========================
    location /api/v1/employee/ {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /employee/ {
        proxy_pass http://127.0.0.1:8080/api/v1/employee/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # =========================
    # ATTENDANCE API
    # =========================
    location /api/v1/attendance/ {
        proxy_pass http://127.0.0.1:8081;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /attendance/ {
        proxy_pass http://127.0.0.1:8081/api/v1/attendance/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # =========================
    # SALARY API
    # =========================
    location /api/v1/salary/ {
        proxy_pass http://127.0.0.1:8082;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /actuator/ {
        proxy_pass http://127.0.0.1:8082/actuator/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # =========================
    # SWAGGER / DOCS
    # =========================
    location /swagger/ {
        proxy_pass http://127.0.0.1:8080/swagger/;
    }

    location /apidocs {
        proxy_pass http://127.0.0.1:8000;
    }

    location /salary-documentation {
        proxy_pass http://127.0.0.1:8082;
    }

    # =========================
    # FRONTEND
    # =========================
    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

---

# 🧠 WHY THIS CONFIG

| Section    | Purpose              |
| ---------- | -------------------- |
| proxy_pass | Route API calls      |
| try_files  | Handle React routing |
| root       | Serve static files   |
| headers    | Preserve client info |

---

# 🔗 Step 5 — Enable Config

```bash
sudo ln -s /etc/nginx/sites-available/frontend /etc/nginx/sites-enabled/
```

---

# 🔍 Step 6 — Test Config

```bash
sudo nginx -t
```

✔ Expected:

```
syntax is ok
test is successful
```

---

# 🔁 Step 7 — Restart NGINX

```bash
sudo systemctl restart nginx
```

---

# 🌐 Step 8 — Access Application

```text
http://<SERVER-IP>
```

---

# ✅ Step 9 — Verification Commands

## 🔹 Frontend

```bash
curl http://localhost
```

## 🔹 Employee API

```bash
curl http://localhost/api/v1/employee/search/all
```

## 🔹 Attendance API

```bash
curl http://localhost/api/v1/attendance/search/all
```

## 🔹 Salary API

```bash
curl http://localhost/api/v1/salary/search/all
```

✔ All should return JSON

---

# 🚨 Troubleshooting Guide

---

## ❌ 500 Internal Server Error

### Check logs:

```bash
sudo tail -f /var/log/nginx/error.log
```

---

## ❌ API Not Working

### Verify backend:

```bash
ss -tunlp | grep -E "8000|8080|8082"
```

---

## ❌ React Routing Not Working

Ensure:

```nginx
try_files $uri $uri/ /index.html;
```

---

## ❌ Changes Not Reflecting

```bash
sudo systemctl reload nginx
```

---

## ❌ Permission Issues

```bash
sudo chown -R www-data:www-data /home/mukesh/frontend/build
```

---

# 🧠 Best Practices

* Use `/api/v1/` prefix for all services
* Never expose backend ports publicly
* Use NGINX as single entry point

---

# 🔐 Optional Enhancements

## HTTPS (SSL)

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx
```

---

# 🚀 Final Outcome

✔ Single entry point (NGINX)
✔ Clean API routing
✔ No CORS issues
✔ Production-ready frontend serving

---

# 🎯 Summary

NGINX acts as:

👉 Reverse Proxy
👉 Static File Server
👉 API Gateway (basic)

---

# 📌 Notes

* All frontend API calls MUST use `/api/v1/...`
* Backend services must be running before NGINX

---

# 🎉 Setup Complete

Your frontend is now fully integrated with backend services via NGINX.
