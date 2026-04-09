# 🚀 OT-Microservices Frontend + NGINX Setup (Production SOP)

> 📦 **Complete, production-grade README for deploying React frontend with NGINX reverse proxy**

---

## 🧩 Overview

This module sets up the **React Frontend + NGINX Reverse Proxy** for OT-Microservices.

It integrates multiple backend services into a **single entry point (port 80)**.

---

## 🏗️ Architecture

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

## 📁 Repository Context

Frontend lives inside project structure:

```
frontend/
├── src/
├── public/
├── package.json
```

Refer full structure here 👉 (root project tree) fileciteturn0file0

---

## ⚙️ Prerequisites

| Component    | Requirement          |
| ------------ | -------------------- |
| OS           | Ubuntu 20.04 / 22.04 |
| Node.js      | v16+                 |
| npm          | v8+                  |
| NGINX        | Installed            |
| Backend APIs | Running              |

---

## 🔍 Step 0 — Verify Backend Services

```bash
curl http://localhost/api/v1/employee/search/all
curl http://localhost/api/v1/attendance/search/all
curl http://localhost/api/v1/salary/search/all
```

✔ Must return JSON

---

## 📦 Step 1 — Clone Frontend

```bash
git clone https://github.com/OT-MICROSERVICES/frontend.git
cd frontend
```

---

## 📦 Step 2 — Install Dependencies

```bash
npm install
```

---

## 🔧 Step 3 — API Standardization (CRITICAL)

All API calls MUST use:

```
/api/v1/...
```

### Verify

```bash
grep -rn "fetch('/" src/
```

---

## 🛠️ Step 4 — Required Code Fixes

👉 Full file changes documented here:

➡️ **Frontend Fixes README** → fileciteturn1file0

### Key Fix Summary

| File              | Action            |
| ----------------- | ----------------- |
| AttendanceList.js | Full replace      |
| EmployeeData.js   | Full replace      |
| ListSalary.js     | Logic updated     |
| HomePage.react.js | Component removed |
| EmployeeForm.js   | API path fixed    |

---

## 🌐 Step 5 — Install & Configure NGINX

### Install

```bash
sudo apt update
sudo apt install nginx -y
```

### Verify

```bash
systemctl status nginx
```

---

## 📁 Step 6 — Build Frontend

```bash
npm run build
```

Verify:

```bash
ls build/
```

---

## ⚙️ Step 7 — NGINX Configuration

```bash
sudo nano /etc/nginx/sites-available/frontend
```

### 🔥 Final Production Config

```nginx
server {
    listen 80;
    server_name _;

    root /home/mukesh/frontend/build;
    index index.html;

    location /api/v1/employee/ {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /api/v1/attendance/ {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /api/v1/salary/ {
        proxy_pass http://127.0.0.1:8082;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

---

## 🔗 Step 8 — Enable Configuration

```bash
sudo ln -s /etc/nginx/sites-available/frontend /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

---

## 🌍 Step 9 — Access Application

```
http://<SERVER-IP>
```

---

## ✅ Validation Checklist

| Check          | Command                                   |
| -------------- | ----------------------------------------- |
| Frontend       | curl [http://localhost](http://localhost) |
| Employee API   | curl /api/v1/employee/search/all          |
| Attendance API | curl /api/v1/attendance/search/all        |
| Salary API     | curl /api/v1/salary/search/all            |

---

# 🚨 Troubleshooting

### ❌ No Data in UI

```bash
grep -rn "fetch('/" src/
```

---

### ❌ Backend Not Reachable

```bash
ss -tunlp | grep -E "8000|8080|8082"
```

---

### ❌ NGINX Error

```bash
sudo tail -f /var/log/nginx/error.log
```

---

### ❌ React Routing Issue

Ensure:

```
try_files $uri $uri/ /index.html;
```

---

### ❌ Permission Issue

```bash
sudo chown -R www-data:www-data ~/frontend/build
```

---

## 🧠 Best Practices

* Use `/api/v1/` consistently
* Do NOT expose backend ports
* Keep NGINX as entry point

---

## 🔐 Optional Enhancements

### Enable HTTPS

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx
```

---

## 🧠 Architecture Notes

* ❌ No API Gateway
* ❌ No service sync
* ✅ Frontend aggregation layer

---

## 🎯 Final Outcome

✔ Unified UI
✔ Reverse proxy working
✔ Microservices integrated
✔ Production-ready deployment

---

## 📌 References

* Project Structure → fileciteturn0file0
* Frontend Fixes → fileciteturn1file0

---

# 🎉 Setup Complete

Your frontend is now fully integrated with backend services using NGINX 🚀
