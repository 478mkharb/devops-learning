# Multistage Dockerfile: Use Case & Why It’s Lighter

## What is a Multistage Dockerfile?

A **multistage Dockerfile** allows you to use **multiple `FROM` statements** in a single Dockerfile.

Each `FROM` starts a **new stage**. You can:

* Build or compile your application in one stage
* Copy only the required output into a final, minimal stage

👉 Result: **smaller, cleaner, and more secure Docker images**.

---

## Why Do We Need Multistage Builds?

Traditional Dockerfiles often:

* Use heavy base images (JDK, build tools, compilers)
* Leave behind temporary files, caches, and dependencies
* Produce **large images** (hundreds of MBs)

Multistage builds solve this by **separating build-time and runtime concerns**.

---

## Key Use Cases

### 1. Compiled Languages

* Java (Maven / Gradle)
* Go
* Rust
* C / C++

You need heavy tools to *build*, but not to *run*.

---

### 2. Frontend Applications

* React / Angular / Vue

Node.js is needed to build assets, but production only needs:

* Nginx
* Static files

---

### 3. Security & Compliance

* Smaller attack surface
* No compilers or package managers in production images

---

### 4. Faster CI/CD & Deployment

* Smaller images → faster pull/push
* Faster container startup

---

## Why Is a Multistage Image Lighter?

Because the **final image contains only what you explicitly copy into it**.

Everything else:

* Build tools
* Source code
* Temporary artifacts

❌ Is discarded automatically.

---

## Example 1: Java Application (Maven)

### ❌ Traditional Dockerfile (Heavy)

```dockerfile
FROM maven:3.9-eclipse-temurin-17
WORKDIR /app
COPY . .
RUN mvn clean package
CMD ["java", "-jar", "target/app.jar"]
```

**Problems:**

* Maven + JDK included in runtime
* Image size ~600–800MB

---

### ✅ Multistage Dockerfile (Optimized)

```dockerfile
# Stage 1: Build
FROM maven:3.9-eclipse-temurin-17 AS builder
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

# Stage 2: Runtime
FROM eclipse-temurin:17-jre
WORKDIR /app
COPY --from=builder /app/target/app.jar app.jar

CMD ["java", "-jar", "app.jar"]
```

**What’s happening?**

* Maven exists only in the `builder` stage
* Final image has only:

  * JRE
  * Compiled JAR

**Final image size:** ~150–200MB

---

## Example 2: Node.js Frontend (React)

```dockerfile
# Stage 1: Build
FROM node:20 AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Serve
FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
```

**Why this is powerful:**

* Node.js is NOT in production
* Nginx Alpine image ~25MB

---

## How Docker Handles Stages Internally

* Each stage is an **intermediate image**
* Only the **last stage** is shipped
* Intermediate stages are cached for faster rebuilds

---

## Best Practices

* Name stages using `AS builder`
* Use minimal runtime images (`alpine`, `distroless`)
* Copy only required artifacts
* Keep build context small (`.dockerignore`)

---

## Quick Comparison

| Feature     | Single Stage | Multistage |
| ----------- | ------------ | ---------- |
| Image Size  | Large        | Small      |
| Security    | Lower        | Higher     |
| CI Speed    | Slower       | Faster     |
| Cleanliness | Messy        | Clean      |

---

## When NOT to Use Multistage?

* Very small scripts
* Quick experiments
* When build == runtime dependencies

---

## Summary

Multistage Dockerfiles:

* Separate **build** and **runtime**
* Reduce image size dramatically
* Improve security and performance

If you want, I can also:

* Convert one of your existing Dockerfiles
* Show Go / Python / .NET examples
* Explain distroless images next
