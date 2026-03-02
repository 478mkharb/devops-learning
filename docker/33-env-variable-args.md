# Why Environment Variables Are Used in Dockerfile and Docker Compose

Environment variables (ENV vars) are a **core mechanism** in Docker for making containerized applications **configurable, portable, secure, and reusable**. They help separate **application code** from **environment-specific configuration**.

---

## 1. What Are Environment Variables in Containers?

Environment variables are **key–value pairs** available to processes running inside a container.

Examples:

* `DB_HOST=db`
* `DB_PORT=5432`
* `APP_ENV=production`

Inside a container, the application reads these values at runtime.

---

## 2. Why Environment Variables Are Needed

### 2.1 Configuration Without Rebuilding Images

Without env vars:

* Any config change → rebuild image ❌

With env vars:

* Same image runs in **dev, test, prod** with different configs ✅

```text
Same Image + Different ENV = Different Behavior
```

---

### 2.2 Separation of Code and Configuration (12‑Factor Principle)

* Image → **application logic**
* Environment variables → **runtime configuration**

This keeps images:

* Clean
* Reusable
* Cloud‑native

---

### 2.3 Security (Secrets Are Not Hard‑Coded)

❌ Bad:

```dockerfile
ENV DB_PASSWORD=mysecret123
```

✅ Better:

* Inject secrets at runtime
* Use `.env`, Docker secrets, or external secret managers

---

## 3. Environment Variables in Dockerfile

### 3.1 Purpose in Dockerfile

ENV in Dockerfile is used to:

* Set **default values**
* Define **image-level configuration**
* Provide variables required by the application

---

### 3.2 Dockerfile Example

```dockerfile
FROM node:20

WORKDIR /app

ENV APP_ENV=production \
    APP_PORT=3000

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE $APP_PORT
CMD ["node", "server.js"]
```

### 3.3 What Happens Here

* `APP_ENV` and `APP_PORT` exist inside the container
* Application can read them using `process.env.APP_ENV`
* Values can be overridden at runtime

---

### 3.4 Overriding Dockerfile ENV at Runtime

```bash
docker run -e APP_ENV=development -e APP_PORT=8080 my-app
```

Runtime values **override Dockerfile defaults**.

---

## 4. Environment Variables in Docker Compose

Docker Compose is **where env vars shine the most**.

---

### 4.1 Why Compose Uses Environment Variables

Compose uses env vars to:

* Configure **multiple services**
* Connect services together
* Avoid hardcoding values
* Change behavior per environment

---

### 4.2 docker-compose.yml Example

```yaml
version: "3.9"

services:
  app:
    image: my-app
    environment:
      APP_ENV: production
      DB_HOST: db
      DB_PORT: 5432
    ports:
      - "8080:3000"
    depends_on:
      - db

  db:
    image: postgres:15
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secret
```

---

### 4.3 How Services Communicate Using ENV Vars

* `DB_HOST=db` → resolves via Docker DNS
* App connects to database using env values
* No IP addresses are hardcoded

---

## 5. Using `.env` File with Docker Compose

### 5.1 Why `.env` Files Are Used

* Avoid repeating values
* Keep secrets out of YAML
* Easy environment switching

---

### 5.2 `.env` File Example

```env
APP_ENV=production
APP_PORT=3000
DB_HOST=db
DB_PORT=5432
DB_USER=admin
DB_PASSWORD=secret
```

### 5.3 docker-compose.yml Using `.env`

```yaml
services:
  app:
    image: my-app
    env_file:
      - .env
    ports:
      - "8080:${APP_PORT}"
```

---

## 6. ENV vs ARG in Dockerfile (Important Distinction)

| Feature                  | ENV     | ARG |
| ------------------------ | ------- | --- |
| Available at runtime     | Yes     | No  |
| Used during build        | Limited | Yes |
| Can be overridden        | Yes     | Yes |
| Visible inside container | Yes     | No  |

Example:

```dockerfile
ARG NODE_VERSION=20
FROM node:${NODE_VERSION}
```

---

## 7. Priority Order of Environment Variables

Highest priority wins:

1. `docker run -e`
2. docker-compose `environment:`
3. docker-compose `env_file`
4. Dockerfile `ENV`

---

## 8. Real‑World Use Cases

### 8.1 Same Image, Multiple Environments

```text
Dev  → APP_ENV=dev
QA   → APP_ENV=qa
Prod → APP_ENV=prod
```

---

### 8.2 Feature Flags

```env
FEATURE_X_ENABLED=true
```

Application behavior changes without rebuild.

---

### 8.3 Database & External Services

* Database credentials
* API endpoints
* Cache hosts

---

## 9. Key Takeaways

* Environment variables make containers **configurable**
* Dockerfile ENV → defaults and image-level config
* Docker Compose ENV → service-level and runtime config
* `.env` files help manage values cleanly
* Same image can run everywhere without changes

---

If you want, I can also add:

* Kubernetes comparison (ConfigMap vs Secret)
* Common interview questions
* Runtime debugging (`env`, `printenv`, `docker inspect`)
