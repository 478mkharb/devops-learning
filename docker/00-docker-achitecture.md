# Docker Architecture – Detailed Explanation

Docker follows a **client–server architecture** where the Docker Client communicates with the Docker Daemon, which manages images, containers, networks, and volumes using the underlying Linux kernel.

---

## 1. High-Level Docker Architecture (Client, Host, Registry Placement)

Docker architecture is best understood by clearly separating **Docker Client**, **Docker Host**, and **Docker Registry**, and then seeing how they interact.

```groovy
+---------------------------+            +----------------------------------+            +---------------------------+
|     Docker Client         |            |          Docker Host             |            |     Docker Registry       |
|  (docker CLI, API calls)  |            |  (where containers run)          |            |  (image storage)          |
|                           |            |                                  |            |                           |
| docker build / run / ps   | --REST-->  |  Docker Daemon (dockerd)         | <--HTTPS-> |  Docker Hub / ECR / GCR   |
|                           |            |     |                            |            |  Private Registry         |
+---------------------------+            |     v                            |            +---------------------------+
                                         |  containerd                      |
                                         |     v                            |
                                         |    runc                          |
                                         |     v                            |
                                         |  Linux Kernel                    |
                                         +----------------------------------+
```

```groovy

┌───────────────────────────────────────────────────────────────────────────┐
│                               USER / CI                                   │
│                                                                           │
│  docker build | docker pull | docker run | docker push                    │
└───────────────────────────────┬───────────────────────────────────────────┘
                                │
                                │ REST API (Unix socket / TCP)
                                ▼
┌───────────────────────────────────────────────────────────────────────────┐
│                         DOCKER CLIENT (CLI)                               │
│                                                                           │
│  - Parses commands                                                        │
│  - Sends API requests                                                     │
└───────────────────────────────┬───────────────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────────────┐
│                         DOCKER HOST (LINUX)                              │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                    DOCKER DAEMON (dockerd)                          │ │
│  │                                                                     │ │
│  │  Manages:                                                           │ │
│  │  Images | Containers | Networks | Volumes                           │ │
│  └───────────────────────────────┬─────────────────────────────────────┘ │
│                                  │ gRPC   (googleRemoteProcedureCall)    │
│                                  ▼                                       │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                       containerd                                    │ │
│  │                                                                     │ │
│  │  - Image unpacking                                                  │ │
│  │  - Snapshot management                                              │ │
│  │  - Container lifecycle execution                                    │ │
│  └───────────────────────────────┬─────────────────────────────────────┘ │
│                                  │ OCI   (OpenContainerInitiative)       │
│                                  ▼                                       │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                            runc                                     │ │
│  │                                                                     │ │
│  │  - Create container process                                         │ │
│  │  - Apply namespaces & cgroups                                       │ │
│  └───────────────────────────────┬─────────────────────────────────────┘ │
│                                  │                                       │
│                                  ▼                                       │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                        LINUX KERNEL                                 │ │
│  │                                                                     │ │
│  │  Namespaces | cgroups | OverlayFS                                   │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
                                │
                                │ HTTPS (pull / push images)
                                ▼
┌───────────────────────────────────────────────────────────────────────────┐
│                           DOCKER REGISTRY                                 │
│                                                                           │
│  Docker Hub | AWS ECR | GCR | Private Registry                            │
│                                                                           │
│  Stores image layers & manifests                                          │
└───────────────────────────────────────────────────────────────────────────┘

```

---

## 2. Docker Client

### What It Is

* Command-line tool (`docker`)
* Used by users to interact with Docker

### What It Does

* Sends commands like:

  * `docker build`
  * `docker pull`
  * `docker run`
  * `docker ps`

### Communication

* Talks to Docker Daemon using **REST API** over:

  * Unix socket: `/var/run/docker.sock`
  * TCP (remote Docker hosts)

---

## 3. Docker Daemon (dockerd)

### What It Is

* Background service running on the host
* Brain of Docker

### Responsibilities

* Builds images
* Pulls images from registries
* Creates and manages containers
* Manages networks and volumes

### Key Point

> Docker Client never touches containers directly — **everything goes through dockerd**

---

## 4. Docker Images

### What Is an Image

* Read-only template
* Built from Dockerfile
* Made of **layers**

### Image Layers

```text
Base OS Layer
Application Dependencies
Application Code
Metadata Layer
```

### Storage

* Stored locally on host
* Cached for reuse

---

## 5. Docker Containers

### What Is a Container

* Running instance of an image
* Image + writable layer

### Characteristics

* Lightweight
* Isolated
* Fast startup

### Writable Layer

* All runtime changes go here
* Deleted when container is removed

---

## 6. Container Runtime (runc)

### Role

* Low-level runtime responsible for:

  * Creating containers
  * Applying isolation

### Standards

* Implements **OCI (Open Container Initiative)** specification

### What runc Uses

* Linux namespaces → isolation
* cgroups → resource limits

---

## 7. Linux Kernel Features Used by Docker

### Namespaces (Isolation)

* PID → process isolation
* NET → network isolation
* MNT → filesystem isolation
* UTS → hostname isolation
* IPC → shared memory isolation
* USER → user isolation

### cgroups (Resource Control)

* CPU limits
* Memory limits
* Disk I/O
* Network bandwidth

---

## 8. Docker Registry

### What It Is

* Storage for Docker images

### Examples

* Docker Hub
* AWS ECR
* GCR
* Private registries

### Image Pull Flow

```text
Docker Client → Docker Daemon → Registry → Image Layers → Local Cache
```

---

## 9. Docker Networking

### Default Networks

* bridge (default)
* host
* none

### How Containers Communicate

* Docker creates virtual Ethernet pairs
* Uses Linux bridge
* Built-in DNS for container name resolution

---

## 10. Docker Volumes

### Purpose

* Persistent storage

### Why Needed

* Containers are ephemeral
* Data must survive restarts

### Types

* Volumes
* Bind mounts
* tmpfs

---

## 11. Full Docker Flow (End-to-End)

### Example: `docker run nginx`

```text
1. User runs: docker run nginx
2. Docker Client sends request to dockerd
3. Docker Daemon checks local images
4. Image not found → pulls from registry
5. Image layers stored locally
6. Daemon creates container metadata
7. runc is invoked
8. Kernel namespaces & cgroups applied
9. Network namespace created
10. Container process starts (nginx)
```

---

## 12. Docker Build Flow

```text
Dockerfile → docker build
   ↓
Each instruction creates a layer
   ↓
Layers cached locally
   ↓
Final image stored
```

---

## 13. Key Takeaways

* Docker uses **client–server architecture**
* Docker Daemon controls everything
* Images are immutable, containers are runtime instances
* runc + kernel provide isolation
* Same architecture works locally and in production

---

This architecture is the **foundation for Kubernetes**, where:

* Docker (or container runtime) runs on nodes
* Kubernetes controls containers via CRI
