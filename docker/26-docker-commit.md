## Goal

Create a **Docker image from a running container** so the current state of the container (installed packages, file changes, configs) is captured into a reusable image.

---

## Core Command

```bash
docker commit <container_id_or_name> <new_image_name>:<tag>
```

This takes a **snapshot** of the container’s filesystem and saves it as a new image.

---

## Step-by-Step Example

### 1️⃣ Run a Container

```bash
docker run -it --name my-nginx ubuntu:22.04 bash
```

You now have a running Ubuntu container.

---

### 2️⃣ Make Changes Inside the Container

```bash
apt update
apt install -y nginx

# create a custom file
echo "Hello from running container" > /var/www/html/index.html
```

Start nginx (optional):

```bash
service nginx start
```

Exit the container:

```bash
exit
```

---

### 3️⃣ Commit the Running Container to an Image

```bash
docker commit my-nginx my-nginx-image:v1
```

What Docker does internally:

* Takes the container’s **writable layer**
* Saves it as a **new image layer**
* Links it to the base image (ubuntu:22.04)

---

### 4️⃣ Verify the Image

```bash
docker images
```

Expected output (example):

```text
REPOSITORY        TAG   IMAGE ID       CREATED          SIZE
my-nginx-image    v1    a1b2c3d4e5      few seconds ago  180MB
```

---

### 5️⃣ Run a Container from the New Image

```bash
docker run -d -p 8080:80 my-nginx-image:v1 nginx -g 'daemon off;'
```

Test:

```bash
curl http://localhost:8080
```

Output:

```text
Hello from running container
```

---

## Visual Flow (Conceptual)

```text
Base Image (ubuntu)
        ↓
Running Container
        ↓  (install nginx, modify files)
Modified Container State
        ↓  docker commit
New Image (my-nginx-image:v1)
        ↓
Reusable Containers
```

---

## Important Notes

* `docker commit` **does NOT**:

  * Capture volumes
  * Capture container runtime configs (ports, env vars, CMD)

* Metadata like CMD/ENTRYPOINT can be added:

```bash
docker commit \
  --change='CMD ["nginx", "-g", "daemon off;"]' \
  my-nginx my-nginx-image:v2
```

---

## When to Use `docker commit`

✔ Debugging / quick experiments
✔ Capturing hotfix changes
✔ Learning & demos

❌ Production builds (use Dockerfile instead)

---

## Best Practice (Production)

* Use **Dockerfile** for reproducibility
* Treat `docker commit` as a **temporary snapshot tool**

---

If you want, I can next:

* Compare `docker commit` vs `Dockerfile`
* Show how Kubernetes uses images created this way
* Convert this flow into an interview-ready answer
