## Overview

Both `docker commit` and `docker export` take data **from a running container**, but they serve **very different purposes**. One creates a **Docker image**, the other extracts a **raw filesystem**.

---

## Core Difference (One Line)

> **`docker commit` → container ➜ Docker image (runnable)**
> **`docker export` → container ➜ filesystem tar (not runnable)**

---

## docker commit — In Detail

### What it does

* Takes the **writable layer** of a running container
* Saves it as a **new image layer**
* Preserves Docker metadata (CMD, ENV, ENTRYPOINT)

### Command

```bash
docker commit <container_name> <image_name>:<tag>
```

---

### Example: docker commit

#### 1️⃣ Run a container

```bash
docker run -it --name web ubuntu:22.04 bash
```

#### 2️⃣ Modify container

```bash
apt update
apt install -y nginx
echo "Hello from commit" > /var/www/html/index.html
exit
```

#### 3️⃣ Commit container to image

```bash
docker commit web web-image:v1
```

#### 4️⃣ Run new image

```bash
docker run -d -p 8080:80 web-image:v1 nginx -g 'daemon off;'
```

Result:

```text
curl http://localhost:8080
Hello from commit
```

---

### Use Cases for docker commit

* Debugging broken containers
* Capturing hotfixes quickly
* Learning and demos
* Temporary or experimental images

---

## docker export — In Detail

### What it does

* Exports the **entire container filesystem**
* Flattens everything into a **single tar archive**
* Drops **all Docker metadata**

### Command

```bash
docker export <container_name> > container_fs.tar
```

---

### Example: docker export

#### 1️⃣ Run a container

```bash
docker run -it --name app ubuntu:22.04 bash
```

#### 2️⃣ Modify container

```bash
apt update
apt install -y curl
echo "export example" > /data.txt
exit
```

#### 3️⃣ Export filesystem

```bash
docker export app > app_fs.tar
```

#### 4️⃣ Inspect tar

```bash
tar -tf app_fs.tar | head
```

Result:

```text
bin/
boot/
data.txt
etc/
usr/
```

---

### Importing back (optional)

```bash
cat app_fs.tar | docker import - app-image:raw
```

⚠ Resulting image:

* No CMD
* No ENV
* No history

---

### Use Cases for docker export

* Filesystem backup
* Container data migration
* Security / forensic analysis
* Creating minimal root filesystem

---

## Side-by-Side Comparison

| Feature            | docker commit | docker export |
| ------------------ | ------------- | ------------- |
| Output             | Docker image  | tar archive   |
| Runnable directly  | Yes           | No            |
| Keeps CMD / ENV    | Yes           | No            |
| Keeps layers       | Yes           | No            |
| Image history      | Yes           | No            |
| Used in Kubernetes | Yes           | No            |

---

## Visual Flow Comparison

```text
 docker commit
 Container
    ↓
 Image (layers + metadata)
    ↓
 docker run / Kubernetes
```

```text
 docker export
 Container
    ↓
 tar (filesystem only)
    ↓
 backup / import / analysis
```

---

## Important Notes

* ❌ Volumes are NOT included in either
* `docker export` flattens all layers
* `docker commit` is NOT reproducible

---

## Production Best Practice

🚫 Avoid both for production builds

✅ Use Dockerfile instead:

```text
Dockerfile → docker build → versioned image
```

---

## Interview-Ready Summary

> `docker commit` creates a runnable Docker image preserving metadata and layers, while `docker export` only extracts a raw filesystem without Docker context. Use commit for debugging, export for filesystem backup. Neither should replace Dockerfile-based builds in production.

