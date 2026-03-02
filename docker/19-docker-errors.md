# 🐳 Common Docker Errors — Detailed Explanation with Examples

This document explains **frequently faced Docker errors**, why they occur, **how to identify the root cause**, and **how to fix them** with practical examples. This is written from a DevOps / production troubleshooting perspective.

---

## 1️⃣ Cannot connect to the Docker daemon

### ❌ Error

```bash
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```

### 🔍 Why it happens

* Docker service is **not running**
* Current user has **no permission** to access Docker socket
* Docker installed but daemon failed to start

### ✅ How to fix

```bash
# Check service status
sudo systemctl status docker

# Start Docker
sudo systemctl start docker

# Add user to docker group
sudo usermod -aG docker $USER
newgrp docker
```

### 🧠 Example

Running `docker ps` immediately after installing Docker without logging out.

---

## 2️⃣ Port is already in use

### ❌ Error

```bash
Error starting userland proxy: listen tcp 0.0.0.0:8080: bind: address already in use
```

### 🔍 Why it happens

* Another container or service is already using the port
* Previous container didn’t stop cleanly

### ✅ How to fix

```bash
# Find process using port
sudo lsof -i :8080

# Stop container using that port
docker ps

docker stop <container_id>
```

### 🧠 Example

```bash
docker run -p 8080:80 nginx
# Another nginx already running on 8080
```

---

## 3️⃣ No space left on device

### ❌ Error

```bash
write /var/lib/docker/... no space left on device
```

### 🔍 Why it happens

* Disk full due to:

  * Old images
  * Stopped containers
  * Dangling volumes

### ✅ How to fix

```bash
# Check disk usage
df -h

# Clean unused Docker data
docker system prune -a

# Remove unused volumes
docker volume prune
```

### 🧠 Example

CI server running Docker builds without cleanup.

---

## 4️⃣ Permission denied on bind mounts

### ❌ Error

```bash
permission denied while trying to connect to the Docker daemon socket
```

OR

```bash
permission denied: /app/data
```

### 🔍 Why it happens

* Host directory permissions mismatch
* SELinux enabled (common in RHEL/CentOS)

### ✅ How to fix

```bash
# Fix permissions
sudo chown -R 1000:1000 /host/path

# For SELinux
chcon -Rt svirt_sandbox_file_t /host/path
```

### 🧠 Example

```bash
docker run -v /root/data:/data nginx
```

Container user cannot access `/root/data`.

---

## 5️⃣ Container restart loop

### ❌ Symptom

```bash
docker ps
# STATUS: Restarting (1) every few seconds
```

### 🔍 Why it happens

* Application crashes on startup
* Wrong command or missing config
* App expects env variable or dependency

### ✅ How to debug

```bash
docker logs <container_id>

docker inspect <container_id>
```

### 🧠 Example

```bash
docker run node-app
# App exits due to missing DB connection
```

---

## 6️⃣ Failed to build Docker image

### ❌ Error

```bash
failed to build image
```

### 🔍 Why it happens

* Syntax error in Dockerfile
* Missing file in COPY
* Network issue during `apt-get`

### ✅ How to fix

```bash
# Build with plain progress
docker build --progress=plain .
```

### 🧠 Example

```Dockerfile
COPY app.jar /app/app.jar
# app.jar does not exist
```

---

## 7️⃣ Error response from daemon: conflict

### ❌ Error

```bash
Error response from daemon: Conflict. The container name "/web" is already in use
```

### 🔍 Why it happens

* Container with same name already exists (running or stopped)

### ✅ How to fix

```bash
docker rm web
# or
docker run --name web2 nginx
```

### 🧠 Example

Re-running `docker run --name web nginx` without deleting old container.

---

## 8️⃣ Docker network not reachable

### ❌ Error

```bash
network mynet not found
```

OR containers can’t communicate

### 🔍 Why it happens

* Network deleted
* Containers on different networks

### ✅ How to fix

```bash
docker network ls

docker network create mynet

docker network connect mynet container1
```

### 🧠 Example

Backend and frontend running on default bridge but expecting custom network.

---

## 9️⃣ Volume mount not working

### ❌ Symptom

* Data not persisting
* Files not visible inside container

### 🔍 Why it happens

* Wrong mount path
* Host directory empty
* File vs directory mismatch

### ✅ How to fix

```bash
# Inspect mounts
docker inspect <container_id>
```

### 🧠 Example

```bash
docker run -v data:/app/data myapp
# Volume exists but app writes to /app/files
```

---

## ✅ Quick Debug Checklist

| Check         | Command                   |
| ------------- | ------------------------- |
| Docker status | `systemctl status docker` |
| Logs          | `docker logs`             |
| Inspect       | `docker inspect`          |
| Disk usage    | `df -h`                   |
| Cleanup       | `docker system prune`     |

---

📌 **Tip:** Most Docker issues are caused by **permissions, ports, disk space, or incorrect paths**. Always start debugging with logs and inspect.
