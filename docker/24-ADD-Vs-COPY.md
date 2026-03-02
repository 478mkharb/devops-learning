# COPY vs ADD in Dockerfile

Both **COPY** and **ADD** are Dockerfile instructions used to copy files into an image. They look similar, but they behave differently and are meant for different use cases.

---

## 1️⃣ COPY — *Simple & Predictable*

### What COPY does

* Copies **local files/directories** from the build context into the image
* Does **exactly one thing**: copy files
* No magic, no side effects

### Syntax

```dockerfile
COPY <src> <dest>
```

### Example

```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```

**What happens:**

* `index.html` from your local directory is copied into the image
* No extraction, no download, no modification

### When to use COPY

* Application source code
* Config files (nginx.conf, app.yaml, etc.)
* Static assets

✅ **Best practice**: Use COPY in most cases

---

## 2️⃣ ADD — *COPY + Extra Features*

### What ADD does

ADD can do everything COPY does **plus**:

1. **Auto-extract local tar archives** (`.tar`, `.tar.gz`, `.tgz`)
2. **Download files from remote URLs**

### Syntax

```dockerfile
ADD <src> <dest>
```

---

### Example 1: Auto-extract tar file

```dockerfile
FROM alpine
ADD app.tar.gz /app/
```

**What happens:**

* `app.tar.gz` is automatically extracted into `/app/`
* You did NOT explicitly ask for extraction

⚠️ This behavior is implicit and sometimes confusing

---

### Example 2: Download from URL

```dockerfile
FROM alpine
ADD https://example.com/file.txt /data/file.txt
```

**What happens:**

* Docker downloads the file during build
* File checksum is NOT verified
* Docker cache becomes unreliable

⚠️ **Not recommended**

---

## 3️⃣ Why ADD is Discouraged

| Reason          | Explanation                      |
| --------------- | -------------------------------- |
| Hidden behavior | Auto-extraction is not obvious   |
| Poor caching    | URL downloads break cache easily |
| Less control    | No checksum validation           |
| Harder to debug | Builds become non-deterministic  |

---

## 4️⃣ Recommended Alternatives

### Instead of ADD for URLs ❌

```dockerfile
ADD https://example.com/app.tar.gz /app
```

### Use RUN + curl/wget ✅

```dockerfile
RUN wget https://example.com/app.tar.gz \
 && tar -xzf app.tar.gz \
 && rm app.tar.gz
```

✔ Explicit
✔ Better caching
✔ Easier debugging

---

## 5️⃣ COPY vs ADD — Side‑by‑Side Comparison

| Feature                   | COPY | ADD |
| ------------------------- | ---- | --- |
| Copy local files          | ✅    | ✅   |
| Extract tar automatically | ❌    | ✅   |
| Download from URL         | ❌    | ✅   |
| Predictable behavior      | ✅    | ❌   |
| Recommended by Docker     | ✅    | ❌   |

---

## 6️⃣ Docker Official Best Practice

> **Always prefer COPY over ADD** unless you specifically need ADD’s extra features.

---

## 7️⃣ Real‑World Best Practice Dockerfile

```dockerfile
FROM node:20-alpine
WORKDIR /app

COPY package.json package-lock.json ./
RUN npm install

COPY . .
CMD ["npm", "start"]
```

✔ Clean
✔ Predictable
✔ Production‑ready

---

## ✅ Final Rule of Thumb

* **90% of the time → use COPY**
* **Only use ADD** if:

  * You *intentionally* want tar auto‑extraction
  * You clearly understand the side effects

---

If you want, I can:

* Convert this into **interview Q&A**
* Add **multi‑stage Dockerfile examples**
* Create a **one‑page revision cheat sheet**
