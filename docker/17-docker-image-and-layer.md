# Difference Between Docker Image and Docker Layer

This document explains the **difference between a Docker Image and a Docker Layer** in a clear, interview-ready way, with examples and key concepts used in real-world Docker usage.

---

## What is a Docker Image?

A **Docker image** is a **read-only template** used to create containers.

### Key Points

* Immutable (cannot be changed)
* Built from multiple layers
* Contains application code, runtime, libraries, and dependencies
* Used as the source to create containers

### Example

```bash
docker pull nginx
```

Here, `nginx` is a **Docker image**.

---

## What is a Docker Layer?

A **Docker layer** is a **filesystem change** that represents the result of a single instruction in a Dockerfile.

### Key Points

* Each Dockerfile instruction creates a layer
* Layers are cached and reusable
* Layers are read-only
* Multiple images can share the same layers

### Example Dockerfile

```dockerfile
FROM ubuntu                 # Layer 1
RUN apt-get update          # Layer 2
RUN apt-get install -y nginx  # Layer 3
COPY index.html /var/www/html/  # Layer 4
```

Each line creates a **new Docker layer**.

---

## Relationship Between Image and Layer (Important)

* A **Docker image is a stack of layers**
* Docker uses a **Union File System** to merge layers
* When a container is created:

  * Image layers remain read-only
  * A new **writable container layer** is added on top

```text
Writable Container Layer
-----------------------
Image Layer 4
Image Layer 3
Image Layer 2
Image Layer 1
```

---

## Key Differences (Comparison Table)

| Aspect      | Docker Image              | Docker Layer                 |
| ----------- | ------------------------- | ---------------------------- |
| Definition  | Complete package          | Individual filesystem change |
| Scope       | Full runnable artifact    | Part of an image             |
| Mutability  | Read-only                 | Read-only                    |
| Created by  | `docker build`            | Each Dockerfile instruction  |
| Reusability | Used to create containers | Shared across images         |
| Example     | `nginx:latest`            | `RUN apt install nginx`      |

---

## Why Docker Layers Matter

* Faster image builds due to caching
* Reduced disk usage through layer sharing
* Faster image pulls (only changed layers are downloaded)

---

## Common Interview Mistakes

* Confusing Docker layers with containers
* Assuming layers are writable
* Mixing up image layers with container writable layer

---

## Interview One-Liner

> A Docker image is a complete, read-only template used to run containers, while a Docker layer is an individual filesystem change that forms part of the image.

---

## Best Practices

* Order Dockerfile instructions to maximize layer caching
* Combine commands where appropriate to reduce layers
* Use multi-stage builds to minimize final image layers
