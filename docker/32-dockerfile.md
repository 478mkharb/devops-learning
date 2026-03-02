# Dockerfile Components — Complete Guide with Examples

A **Dockerfile** is a declarative script that defines **how a Docker image is built**. Each instruction creates an **image layer** and contributes to the final runtime behavior of the container.

This document explains **every major Dockerfile component**, what it does internally, and when to use it — with real examples.

---

## 1. FROM (Base Image)

### Purpose

* Defines the **starting point** of the image
* Mandatory (except for `scratch` images)

### Syntax

```dockerfile
FROM ubuntu:22.04
```

### Key Points

* First instruction in most Dockerfiles
* Determines:

  * OS libraries
  * Package manager
  * Image size

### Example

```dockerfile
FROM node:20-alpine
```

Lightweight base image → faster pulls, smaller images.

---

## 2. LABEL (Metadata)

### Purpose

* Adds metadata to the image
* Used for documentation, ownership, automation

### Syntax

```dockerfile
LABEL maintainer="devops@example.com"
LABEL version="1.0"
```

### Best Practice

```dockerfile
LABEL org.opencontainers.image.source="https://github.com/org/repo"
```

---

## 3. RUN (Build-time Command)

### Purpose

* Executes commands **during image build**
* Creates a new image layer

### Syntax

```dockerfile
RUN apt-get update && apt-get install -y nginx
```

### What Happens Internally

```
RUN command
  ↓
Temporary container
  ↓
Filesystem changes
  ↓
Committed as image layer
```

### Best Practice

* Combine commands to reduce layers

```dockerfile
RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*
```

---

## 4. COPY (Copy Files from Build Context)

### Purpose

* Copies files from local machine to image

### Syntax

```dockerfile
COPY app.js /app/app.js
```

### Key Points

* Faster than `ADD`
* Only copies local files

---

## 5. ADD (Copy + Extra Features)

### Purpose

* Similar to COPY but with extras

### Extra Capabilities

* Auto-extracts tar archives
* Can download remote URLs

### Syntax

```dockerfile
ADD app.tar.gz /app/
```

### Best Practice

> Prefer `COPY` unless you explicitly need `ADD` features

---

## 6. WORKDIR (Working Directory)

### Purpose

* Sets working directory for subsequent instructions

### Syntax

```dockerfile
WORKDIR /app
```

### Benefits

* Avoids repeated `cd` commands
* Creates directory if not exists

---

## 7. ENV (Environment Variables)

### Purpose

* Sets environment variables in image

### Syntax

```dockerfile
ENV NODE_ENV=production
```

### Runtime Availability

* Available at container runtime
* Can be overridden via `docker run -e`

---

## 8. ARG (Build-time Variables)

### Purpose

* Variables available **only during build**

### Syntax

```dockerfile
ARG VERSION=1.0
```

### Difference: ARG vs ENV

| ARG                    | ENV               |
| ---------------------- | ----------------- |
| Build-time only        | Runtime available |
| Not in final container | Persist in image  |

---

## 9. EXPOSE (Documentation for Ports)

### Purpose

* Documents which port the app listens on

### Syntax

```dockerfile
EXPOSE 8080
```

### Important

* Does NOT publish the port
* Actual publishing done via `-p`

---

## 10. USER (Non-root Execution)

### Purpose

* Runs container as non-root user

### Syntax

```dockerfile
USER node
```

### Why Important

* Improves security
* Prevents container breakout risks

---

## 11. VOLUME (Persistent Data)

### Purpose

* Declares mount point for external storage

### Syntax

```dockerfile
VOLUME /data
```

### Behavior

* Docker creates anonymous volume if not specified

---

## 12. CMD (Default Runtime Command)

### Purpose

* Defines default command when container starts

### Syntax

```dockerfile
CMD ["node", "app.js"]
```

### Overridable

```bash
docker run image echo "hello"
```

---

## 13. ENTRYPOINT (Main Container Process)

### Purpose

* Defines **fixed executable** for container

### Syntax

```dockerfile
ENTRYPOINT ["nginx", "-g", "daemon off;"]
```

### ENTRYPOINT vs CMD

| ENTRYPOINT       | CMD               |
| ---------------- | ----------------- |
| Fixed            | Default args      |
| Hard to override | Easily overridden |

---

## 14. HEALTHCHECK (Container Health)

### Purpose

* Allows Docker to monitor container health

### Syntax

```dockerfile
HEALTHCHECK CMD curl -f http://localhost || exit 1
```

### States

* healthy
* unhealthy

---

## 15. ONBUILD (Triggered by Child Images)

### Purpose

* Runs instructions when image is used as base

### Syntax

```dockerfile
ONBUILD COPY . /app
```

---

## 16. STOPSIGNAL (Graceful Shutdown)

### Purpose

* Defines signal sent to PID 1

### Syntax

```dockerfile
STOPSIGNAL SIGTERM
```

---

## 17. Complete Example Dockerfile

```dockerfile
FROM node:20-alpine

LABEL maintainer="devops@example.com"

WORKDIR /app

COPY package*.json ./

RUN npm install --production

COPY . .

ENV NODE_ENV=production

EXPOSE 3000

USER node

CMD ["node", "server.js"]
```

---

## 18. Dockerfile Build Flow

```
Dockerfile
   ↓
Each instruction
   ↓
New image layer
   ↓
Final image
```

---

## 19. Key Best Practices

* Use minimal base images
* Combine RUN instructions
* Use `.dockerignore`
* Avoid running as root
* Prefer COPY over ADD
* Keep CMD simple

---

## 20. Key Takeaways

* Dockerfile defines image, not container
* Each instruction creates a layer
* CMD/ENTRYPOINT define runtime behavior
* Security and size depend on Dockerfile quality

---

