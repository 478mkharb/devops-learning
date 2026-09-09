# Docker Interview Preparation — DevOps

A structured Docker interview-preparation guide covering fundamentals, architecture, internals, images, Dockerfiles, networking, storage, registries, resource management, troubleshooting, orchestration, and scenario-based questions.

The material is organized from **fundamentals → implementation details → production usage → troubleshooting → scenario questions**.

---

## Table of Contents

1. [Docker Fundamentals](#1-docker-fundamentals)
2. [Docker Architecture](#2-docker-architecture)
3. [Docker Internals](#3-docker-internals)
4. [Images, Layers, Tags and Digests](#4-images-layers-tags-and-digests)
5. [Dockerfile](#5-dockerfile)
6. [RUN vs CMD vs ENTRYPOINT](#6-run-vs-cmd-vs-entrypoint)
7. [COPY vs ADD](#7-copy-vs-add)
8. [ARG vs ENV](#8-arg-vs-env)
9. [Multistage Builds and Image Optimization](#9-multistage-builds-and-image-optimization)
10. [Container Lifecycle and Commands](#10-container-lifecycle-and-commands)
11. [Docker Networking](#11-docker-networking)
12. [Docker Storage](#12-docker-storage)
13. [Environment Variables and Configuration](#13-environment-variables-and-configuration)
14. [Container Registries](#14-container-registries)
15. [CPU, Memory and Resource Management](#15-cpu-memory-and-resource-management)
16. [Docker Compose](#16-docker-compose)
17. [Docker Swarm and Orchestration](#17-docker-swarm-and-orchestration)
18. [Container Security and Best Practices](#18-container-security-and-best-practices)
19. [Troubleshooting](#19-troubleshooting)
20. [Exit Codes](#20-exit-codes)
21. [Advanced Docker Topics](#21-advanced-docker-topics)
22. [Docker vs Kubernetes](#22-docker-vs-kubernetes)
23. [Scenario-Based Interview Questions](#23-scenario-based-interview-questions)
24. [Rapid-Fire Questions](#24-rapid-fire-questions)
25. [Important Command Cheat Sheet](#25-important-command-cheat-sheet)
26. [Common Interview Traps](#26-common-interview-traps)
27. [Final Interview Checklist](#27-final-interview-checklist)

---

# 1. Docker Fundamentals

## Q1. What is Docker?

Docker is a containerization platform used to package an application together with its dependencies into a portable image that can run as an isolated container.

### Why Docker?

| Problem | Docker solution |
|---|---|
| "Works on my machine" | Packages application dependencies |
| Environment inconsistency | Same image can run across environments |
| Slow VM startup | Containers share the host kernel |
| Large application packages | Layered images and reusable base layers |
| Manual deployment | Images can be built and deployed consistently |

---

## Q2. What is a container?

A container is an isolated process running from an image.

A useful mental model is:

```text
Docker Image
     |
     | docker run
     v
Container
     |
     v
Running application process
```

A container is **not a lightweight VM**. It is an isolated process environment that uses the host operating system kernel.

---

## Q3. Container vs Virtual Machine

| Feature | Container | Virtual Machine |
|---|---|---|
| Virtualizes | OS-level processes | Hardware/complete OS |
| Kernel | Shares host kernel | Guest has its own kernel |
| Startup | Usually seconds or less | Usually slower |
| Size | Usually MBs/GBs | Usually GBs |
| Isolation | Process/namespace based | Stronger hardware/OS boundary |
| Resource overhead | Lower | Higher |
| Typical use | Microservices, CI/CD | Full OS workloads |

### Interview point

Do not say:

> "A container has its own kernel."

A normal Linux container **shares the host Linux kernel**.

---

## Q4. Why do containers start quickly?

Containers do not normally boot a complete guest operating system. Docker starts an isolated process using Linux kernel features such as namespaces and cgroups.

```text
VM:
Hardware
  ↓
Hypervisor
  ↓
Guest OS boot
  ↓
Application

Container:
Host Kernel
  ↓
Container Runtime
  ↓
Application Process
```

---

## Q5. Why are containers lightweight?

Because containers share the host kernel and generally contain only the application, runtime libraries, configuration and required filesystem content.

---

## Q6. What is the difference between an image and a container?

| Image | Container |
|---|---|
| Read-only template | Running/created instance |
| Immutable by design | Has writable container layer |
| Used to create containers | Created from an image |
| Can be pushed to registry | Usually not pushed as a runtime object |

Example:

```bash
docker pull nginx
docker run -d --name web nginx
```

Here:

```text
nginx image
    ↓
web container
```

---

# 2. Docker Architecture

## Q7. Explain Docker architecture.

A simplified architecture is:

```text
+----------------------+
| Docker CLI / Client  |
+----------+-----------+
           |
           | API
           v
+----------------------+
| Docker Daemon        |
| dockerd              |
+----------+-----------+
           |
           v
+----------------------+
| containerd           |
+----------+-----------+
           |
           v
+----------------------+
| runc / OCI runtime   |
+----------+-----------+
           |
           v
+----------------------+
| Linux Kernel         |
| namespaces + cgroups|
+----------------------+
```

---

## Q8. What is Docker Client?

The Docker Client is the command-line interface through which users send commands to Docker.

Examples:

```bash
docker ps
docker build .
docker run nginx
docker stop web
```

The client communicates with the Docker daemon through the Docker API.

---

## Q9. What is Docker Daemon?

`dockerd` is the Docker daemon. It receives API requests and manages Docker objects such as:

- Images
- Containers
- Networks
- Volumes

---

## Q10. What is the Docker socket?

On a typical Linux installation, Docker CLI communicates with the daemon through:

```text
/var/run/docker.sock
```

Example:

```bash
ls -l /var/run/docker.sock
```

Giving unrestricted access to the Docker socket is security-sensitive because access to the daemon can effectively provide extensive control over the host.

---

## Q11. What happens internally when you run `docker run nginx`?

A simplified flow:

```text
docker run nginx
      ↓
Docker CLI
      ↓
Docker daemon
      ↓
Check local image
      ↓
Pull image if missing
      ↓
Create container
      ↓
Configure filesystem/network
      ↓
Runtime creates isolated process
      ↓
nginx process starts
```

---

# 3. Docker Internals

## Q12. What are Linux namespaces?

Namespaces provide isolation of system resources.

Common namespaces include:

| Namespace | Isolation |
|---|---|
| PID | Process IDs |
| NET | Network interfaces/routes |
| MNT | Mount points |
| UTS | Hostname/domain name |
| IPC | IPC resources |
| USER | User/group IDs |

---

## Q13. What are cgroups?

Control groups provide resource accounting and limiting.

They can control resources such as:

- CPU
- Memory
- PIDs
- Block I/O

Example:

```bash
docker run --memory=512m --cpus=1 nginx
```

This does not create a VM. It applies resource controls to the container's processes.

---

## Q14. What is `runc`?

`runc` is an OCI-compliant low-level container runtime commonly used in the Docker/container ecosystem to create and run containers according to OCI specifications.

---

## Q15. What is containerd?

`containerd` is a container runtime component responsible for container lifecycle and image-related operations. Docker Engine uses containerd as part of its runtime architecture.

---

## Q16. Docker vs containerd vs runc

| Component | Main responsibility |
|---|---|
| Docker CLI | User-facing commands |
| dockerd | Docker Engine API/orchestration of Docker objects |
| containerd | Container lifecycle and image management |
| runc | Low-level OCI container execution |
| Linux kernel | Isolation and resource-control primitives |

---

## Q17. What is PID 1 inside a container?

The first process in a container normally has PID 1 within that container's PID namespace.

PID 1 has special responsibilities around signal handling and child-process reaping.

This is one reason process design matters in containers.

---

# 4. Images, Layers, Tags and Digests

## Q18. What is a Docker image?

A Docker image is an immutable, layered package containing the filesystem and metadata needed to create a container.

---

## Q19. What is a Docker layer?

Each applicable Dockerfile instruction can contribute a filesystem layer.

Example:

```dockerfile
FROM ubuntu:24.04
RUN apt-get update && apt-get install -y nginx
COPY index.html /var/www/html/
```

Conceptually:

```text
Base image layer
       +
Package installation layer
       +
Application layer
       =
Final image
```

---

## Q20. Why are Docker images layered?

Layers allow reuse and caching.

If multiple images use the same base image, that base content does not have to be independently rebuilt for every image.

---

## Q21. What is Docker build cache?

Docker can reuse previous build results when an instruction and its relevant inputs have not changed.

Example:

```dockerfile
COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build
```

This arrangement can preserve the dependency-installation cache when only application source files change.

---

## Q22. What is a tag?

A tag is a human-readable reference such as:

```text
myapp:1.4.0
```

Tags are mutable references.

---

## Q23. What is an image digest?

A digest identifies image content using a cryptographic content address.

Example:

```text
nginx@sha256:<digest>
```

A digest is preferable when you need an immutable content reference.

---

## Q24. Tag vs Digest

| Tag | Digest |
|---|---|
| Human-readable | Content-addressed |
| Can move to different image content | Identifies exact content |
| Convenient for development | Better for immutable deployments |

---

# 5. Dockerfile

## Q25. What is a Dockerfile?

A Dockerfile is a text file containing instructions used to build a Docker image.

Example:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "app.py"]
```

Build:

```bash
docker build -t my-python-app:1.0 .
```

Run:

```bash
docker run --rm my-python-app:1.0
```

---

## Q26. What does `FROM` do?

Defines the base image.

```dockerfile
FROM ubuntu:24.04
```

A Dockerfile generally starts with `FROM` unless using a special syntax such as a scratch-only stage.

---

## Q27. What does `RUN` do?

Executes a command during image build.

```dockerfile
RUN apt-get update && apt-get install -y curl
```

The result becomes part of the image.

---

## Q28. What does `CMD` do?

Defines the default command/arguments used when a container starts.

```dockerfile
CMD ["nginx", "-g", "daemon off;"]
```

CMD can be overridden by supplying another command to `docker run`.

---

## Q29. What does `ENTRYPOINT` do?

Defines the executable that the container is intended to run.

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

Running:

```bash
docker run myapp
```

results conceptually in:

```text
python app.py
```

---

# 6. RUN vs CMD vs ENTRYPOINT

## Q30. Explain RUN vs CMD vs ENTRYPOINT.

| Instruction | When it runs | Purpose |
|---|---|---|
| RUN | Image build time | Install/build/configure |
| CMD | Container runtime | Default command/arguments |
| ENTRYPOINT | Container runtime | Main executable |
| COPY | Image build time | Copy files |
| ADD | Image build time | Copy files plus additional behavior |
| ENV | Image/runtime configuration | Set environment variables |
| ARG | Image build time | Build-time variable |
| WORKDIR | Build/runtime | Set working directory |

### Example

```dockerfile
FROM alpine:3.20

RUN apk add --no-cache curl

ENTRYPOINT ["curl"]
CMD ["https://example.com"]
```

---

## Q31. CMD vs ENTRYPOINT

### CMD only

```dockerfile
CMD ["sleep", "1000"]
```

Can easily be replaced:

```bash
docker run image echo hello
```

### ENTRYPOINT + CMD

```dockerfile
ENTRYPOINT ["sleep"]
CMD ["1000"]
```

Now:

```bash
docker run image 500
```

effectively runs:

```text
sleep 500
```

---

## Q32. Shell form vs exec form

### Shell form

```dockerfile
CMD python app.py
```

### Exec form

```dockerfile
CMD ["python", "app.py"]
```

For production containers, exec form is often preferable because it avoids an extra shell and generally gives cleaner process/signal behavior.

---

# 7. COPY vs ADD

## Q33. COPY vs ADD

| COPY | ADD |
|---|---|
| Primarily copies files/directories | Copy plus additional features |
| Simpler | More behavior |
| Preferred for ordinary copying | Use when ADD-specific behavior is actually required |

Example:

```dockerfile
COPY app.py /app/
```

Prefer `COPY` when you simply need to copy application files.

---

# 8. ARG vs ENV

## Q34. What is ARG?

`ARG` defines a build-time variable.

```dockerfile
ARG APP_VERSION=1.0
RUN echo "Building $APP_VERSION"
```

Build:

```bash
docker build --build-arg APP_VERSION=2.0 -t myapp:2.0 .
```

---

## Q35. What is ENV?

`ENV` defines an environment variable available to subsequent image build instructions and containers created from the image.

```dockerfile
ENV APP_ENV=production
```

Override at runtime:

```bash
docker run -e APP_ENV=staging myapp
```

---

## Q36. ARG vs ENV

| ARG | ENV |
|---|---|
| Build-time variable | Runtime environment variable |
| Not automatically available to running container | Available to container |
| Set with `--build-arg` | Set/override with `-e` |
| Useful for build configuration | Useful for application configuration |

Do not use either as a safe secret-storage mechanism. Secrets should be handled using appropriate secret-management mechanisms.

---

# 9. Multistage Builds and Image Optimization

## Q37. What is a multistage build?

A multistage Dockerfile uses multiple `FROM` stages so build dependencies can be separated from the final runtime image.

Example:

```dockerfile
FROM golang:1.24 AS builder

WORKDIR /src
COPY . .
RUN go build -o app .

FROM debian:bookworm-slim

WORKDIR /app
COPY --from=builder /src/app .
CMD ["./app"]
```

The final image does not need the complete Go toolchain.

---

## Q38. Why use multistage builds?

| Without multistage | With multistage |
|---|---|
| Compiler in final image | Compiler excluded |
| Larger attack surface | Smaller runtime surface |
| Larger image | Smaller image |
| More unnecessary packages | Runtime contains only required components |

---

## Q39. How do you optimize Docker images?

Key practices:

1. Use an appropriate minimal base image.
2. Use multistage builds.
3. Add `.dockerignore`.
4. Order Dockerfile instructions for cache reuse.
5. Remove package-manager caches where appropriate.
6. Install only required packages.
7. Avoid copying unnecessary files.
8. Pin versions where reproducibility matters.
9. Use image scanning.
10. Prefer immutable image references for production deployment.

---

## Q40. What is `.dockerignore`?

`.dockerignore` excludes files from the Docker build context.

Example:

```text
.git
node_modules
__pycache__
*.log
.env
terraform.tfstate
```

This can improve build performance and prevent unnecessary or sensitive files from entering the build context.

---

# 10. Container Lifecycle and Commands

## Q41. Explain the container lifecycle.

```text
Image
  ↓
docker create
  ↓
Created
  ↓
docker start
  ↓
Running
  ↓
docker stop
  ↓
Stopped
  ↓
docker rm
  ↓
Removed
```

`docker run` normally combines creation and startup.

---

## Q42. `docker run` vs `docker start`

| Command | Meaning |
|---|---|
| `docker run` | Creates a new container from an image and starts it |
| `docker start` | Starts an existing stopped container |

---

## Q43. `docker stop` vs `docker kill`

| Command | Behavior |
|---|---|
| `docker stop` | Graceful stop attempt, then force after timeout |
| `docker kill` | Sends a kill signal immediately by default |

---

## Q44. `docker exec` vs `docker attach`

| `docker exec` | `docker attach` |
|---|---|
| Starts a new process in a running container | Attaches to the existing main process |
| Good for debugging | Connects to main process I/O |
| Example: shell | Can affect the attached process/session |

Example:

```bash
docker exec -it web /bin/sh
```

---

## Q45. How do you inspect a container?

```bash
docker inspect web
```

Useful for checking:

- IP address
- Mounts
- Environment
- Network settings
- Entrypoint
- Command
- State
- Resource configuration

---

## Q46. How do you view container logs?

```bash
docker logs web
docker logs -f web
```

---

# 11. Docker Networking

## Q47. What are common Docker network drivers?

| Network | Description |
|---|---|
| bridge | Common isolated container networking on one host |
| host | Container uses host network namespace |
| none | No normal network connectivity |
| overlay | Multi-host networking, commonly associated with orchestration |

---

## Q48. What is a bridge network?

A bridge network allows containers on the same Docker host to communicate through a virtual network.

User-defined bridge networks are preferred for application isolation and service communication.

---

## Q49. How do you create a custom network?

```bash
docker network create app-net

docker run -d --name db --network app-net postgres
docker run -d --name api --network app-net my-api
```

Containers attached to the same user-defined network can communicate using container/service names where Docker's embedded DNS applies.

---

## Q50. What is port mapping?

Example:

```bash
docker run -d -p 8080:80 nginx
```

Means:

```text
Host port 8080
      ↓
Container port 80
```

---

## Q51. Does `EXPOSE` publish a port?

No.

```dockerfile
EXPOSE 80
```

documents the intended container port. It does not by itself publish the port to the host.

Publishing is done, for example, with:

```bash
docker run -p 8080:80 nginx
```

---

## Q52. Why does localhost cause Docker networking problems?

`localhost` means the current network namespace.

Inside a container:

```text
localhost → that container
```

On the host:

```text
localhost → host
```

Therefore, if an application in container A tries:

```text
localhost:5432
```

it is trying to reach port 5432 **inside container A**, not another container or the host.

---

## Q53. How do you troubleshoot Docker networking?

```bash
docker ps
docker port <container>
docker inspect <container>
docker network ls
docker network inspect <network>
docker exec -it <container> sh
```

Then test:

```bash
curl http://service:8080
```

or:

```bash
nc -vz service 8080
```

---

# 12. Docker Storage

## Q54. Why is container filesystem data considered ephemeral?

A container has a writable layer above the image layers. When the container is removed, that writable layer is removed.

Persistent application data should therefore use externalized storage such as Docker volumes or bind mounts.

---

## Q55. What is a Docker volume?

A Docker volume is Docker-managed persistent storage.

```bash
docker volume create pgdata

docker run -d \
  --name postgres \
  -v pgdata:/var/lib/postgresql/data \
  postgres
```

---

## Q56. What is a bind mount?

A bind mount maps a host filesystem path into the container.

```bash
docker run -d \
  -v /opt/myapp/config:/app/config \
  myapp
```

---

## Q57. Volume vs bind mount

| Feature | Volume | Bind mount |
|---|---|---|
| Managed by | Docker | Host filesystem |
| Host path control | Abstracted | Explicit |
| Portability | Usually better | More host-dependent |
| Development use | Good | Very common |
| Persistent application data | Common choice | Also possible |

---

## Q58. What happens to volume data when a container is removed?

A named volume normally remains until explicitly removed.

```bash
docker rm postgres
```

does not normally remove:

```text
pgdata
```

You can inspect:

```bash
docker volume ls
docker volume inspect pgdata
```

---

# 13. Environment Variables and Configuration

## Q59. How do you pass an environment variable to a container?

```bash
docker run -e APP_ENV=production myapp
```

Multiple variables:

```bash
docker run \
  -e APP_ENV=production \
  -e DB_HOST=db \
  myapp
```

---

## Q60. Why should secrets not be hard-coded in Dockerfiles?

Dockerfile instructions and image history/build metadata can expose values. Also, anything embedded into an image can be difficult to rotate.

Bad:

```dockerfile
ENV DB_PASSWORD=supersecret
```

Prefer runtime secret-management mechanisms appropriate to your platform.

---

# 14. Container Registries

## Q61. What is a container registry?

A registry stores and distributes container images.

Examples include:

- Docker Hub
- Amazon ECR
- GitHub Container Registry
- Private enterprise registries

---

## Q62. Explain image tagging and pushing.

```bash
docker build -t myapp:1.0 .

docker tag myapp:1.0 registry.example.com/team/myapp:1.0

docker push registry.example.com/team/myapp:1.0
```

---

## Q63. What happens during `docker push`?

Conceptually:

```text
Local image
    ↓
Check registry authentication
    ↓
Compare existing layers
    ↓
Upload missing layers
    ↓
Upload manifest
    ↓
Tag points to image manifest
```

Already-present layers can generally be reused.

---

## Q64. What is Amazon ECR used for?

Amazon Elastic Container Registry (ECR) is an AWS-managed container registry used to store and distribute container images.

A typical CI/CD flow is:

```text
Developer
   ↓
Git
   ↓
Jenkins / CI
   ↓
docker build
   ↓
Security scan
   ↓
docker push
   ↓
ECR
   ↓
Deployment platform
```

---

# 15. CPU, Memory and Resource Management

## Q65. How do you limit memory?

```bash
docker run --memory=512m myapp
```

---

## Q66. How do you limit CPU?

For example:

```bash
docker run --cpus=1 myapp
```

This applies CPU resource controls rather than creating a dedicated CPU core.

---

## Q67. How do you monitor container resources?

```bash
docker stats
```

For one container:

```bash
docker stats web
```

---

## Q68. What happens when a container exceeds its memory limit?

Depending on the circumstances, processes can be killed by the kernel due to memory pressure/OOM behavior.

A common Docker symptom is:

```text
Exited (137)
```

which often indicates SIGKILL, frequently due to an OOM condition, although the exact cause should be verified.

---

## Q69. How do you clean Docker disk usage?

Start with:

```bash
docker system df
docker system df -v
```

Then selectively clean unused objects:

```bash
docker container prune
docker image prune
docker network prune
docker volume prune
```

Be careful with:

```bash
docker system prune -a --volumes
```

because it can remove unused images, containers, networks and volumes.

---

# 16. Docker Compose

## Q70. What is Docker Compose?

Docker Compose defines and runs multi-container applications using a declarative YAML file.

Example:

```yaml
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: example

  api:
    build: .
    depends_on:
      - db
    ports:
      - "8080:8080"
```

---

## Q71. Why use Compose?

| Without Compose | With Compose |
|---|---|
| Many `docker run` commands | One declarative file |
| Manual networking | Service networking |
| Harder configuration | Centralized YAML |
| Harder local reproduction | Reproducible development environment |

---

## Q72. Does `depends_on` guarantee application readiness?

No.

`depends_on` primarily controls startup ordering. It does not automatically mean the dependent service is ready to accept application traffic.

Use health checks and application-level retry/readiness logic where required.

---

# 17. Docker Swarm and Orchestration

## Q73. Can Docker scale containers?

Yes. You can run multiple instances manually or use an orchestration platform.

For example, Compose can scale services in supported configurations, while Swarm provides native orchestration features.

---

## Q74. What is Docker Swarm?

Docker Swarm is Docker's native orchestration technology for clustering and scheduling containers.

Concepts include:

- Manager nodes
- Worker nodes
- Services
- Tasks
- Desired state

---

## Q75. Docker vs Docker Swarm vs Kubernetes

| Capability | Docker Engine | Docker Swarm | Kubernetes |
|---|---|---|---|
| Container runtime/engine | Yes | Uses Docker ecosystem | Uses CRI-compatible runtimes |
| Single-host containers | Yes | Yes | Yes, but designed for clusters |
| Cluster orchestration | No | Yes | Yes |
| Self-healing | Basic restart policies | Yes | Yes |
| Advanced scheduling | Limited | Yes | Extensive |
| Ecosystem | Docker | Smaller | Very large |
| Typical enterprise orchestration | Less common | Less common | Very common |

---

# 18. Container Security and Best Practices

## Q76. Why should containers avoid running as root?

Running as root inside a container increases the impact of a container compromise.

Prefer a dedicated non-root user:

```dockerfile
FROM python:3.12-slim

RUN useradd --create-home appuser
WORKDIR /app

COPY . .
RUN chown -R appuser:appuser /app

USER appuser

CMD ["python", "app.py"]
```

---

## Q77. What are important Docker security practices?

1. Use trusted base images.
2. Keep images patched.
3. Scan images for vulnerabilities.
4. Avoid unnecessary packages.
5. Run as non-root where possible.
6. Do not bake secrets into images.
7. Minimize Linux capabilities.
8. Use read-only filesystems where practical.
9. Restrict Docker socket access.
10. Use immutable image references for production.
11. Keep the Docker daemon and host patched.
12. Apply least privilege.

---

## Q78. Why is mounting `/var/run/docker.sock` dangerous?

The Docker socket provides access to the Docker daemon. A process with sufficiently powerful access to the daemon may be able to create privileged containers and access host resources.

Therefore, treat Docker socket access as highly privileged.

---

# 19. Troubleshooting

## Q79. A container exits immediately. What do you check?

Start with:

```bash
docker ps -a
docker logs <container>
docker inspect <container>
```

Then check:

```text
Exit code
   ↓
CMD / ENTRYPOINT
   ↓
Application logs
   ↓
Environment variables
   ↓
Configuration
   ↓
Required files
   ↓
Dependencies
```

---

## Q80. Container is restarting continuously. How do you troubleshoot?

```bash
docker ps
docker logs --tail 200 <container>
docker inspect <container>
docker inspect --format '{{.State.ExitCode}}' <container>
```

Check:

- Application crash
- Incorrect command
- Missing configuration
- Healthcheck behavior
- Dependency availability
- Resource limits
- Restart policy

---

## Q81. Application works on host but not inside container.

Investigate:

```text
Application bind address
        ↓
Port mapping
        ↓
Container network
        ↓
Firewall
        ↓
Service name/DNS
        ↓
Environment configuration
```

A common issue is that the application is bound to an address inappropriate for the intended container networking.

---

## Q82. Container cannot connect to another container.

Check:

```bash
docker network ls
docker network inspect app-net
docker inspect api
docker inspect db
```

Then verify:

- Both containers are attached to the expected network.
- Correct service/container name is used.
- Correct port is used.
- Application is actually listening.
- Network policy/firewall constraints are not blocking traffic.

---

## Q83. Container cannot access the internet.

Check:

```bash
docker exec -it <container> sh
```

Then:

```bash
ip addr
ip route
cat /etc/resolv.conf
```

Also check:

- Docker bridge networking
- Host connectivity
- DNS
- Proxy configuration
- Firewall rules
- NAT/forwarding configuration

---

## Q84. Port mapping is not working.

For:

```bash
docker run -d -p 8080:80 nginx
```

check:

```bash
docker ps
docker port <container>
docker inspect <container>
curl http://localhost:8080
```

Remember:

```text
8080 = host port
80   = container port
```

---

## Q85. You receive `permission denied` inside the container.

Possible causes:

- File ownership
- Container user
- Bind-mount permissions
- Linux filesystem permissions
- Read-only mount
- Security controls

Investigate:

```bash
docker exec -it <container> sh
id
ls -la
mount
```

---

## Q86. Docker daemon is not reachable.

Check:

```bash
systemctl status docker
journalctl -u docker
docker info
```

Possible causes:

- Daemon stopped
- Socket permissions
- Docker context
- Configuration problem
- Disk/resource issue

---

## Q87. `docker` works with sudo but not without sudo.

Check:

```bash
ls -l /var/run/docker.sock
groups
```

On systems configured for Docker group access, the user may need appropriate membership and a refreshed login/session.

Do not casually grant broad privileged access without understanding the security implications.

---

## Q88. Disk usage is unexpectedly high.

Start with:

```bash
docker system df -v
```

Then investigate:

```bash
docker images
docker ps -a
docker volume ls
```

Also inspect host filesystem usage:

```bash
df -h
du -sh /var/lib/docker
```

Avoid deleting Docker data directories manually while Docker is running.

---

# 20. Exit Codes

## Q89. What does exit code 0 mean?

The process completed successfully.

---

## Q90. What does exit code 1 usually mean?

Generic application-level failure. The exact meaning depends on the application.

---

## Q91. What does exit code 125 mean?

Docker itself generally failed to run the container command.

---

## Q92. What does exit code 126 mean?

The command was found but could not be executed, often due to permissions or executable format.

---

## Q93. What does exit code 127 mean?

The command was not found.

---

## Q94. What does exit code 137 mean?

137 is commonly:

```text
128 + 9
```

where signal 9 is `SIGKILL`.

A frequent cause is an OOM kill, but verify the actual cause rather than assuming.

---

## Exit Code Quick Table

| Exit code | Common interpretation |
|---:|---|
| 0 | Success |
| 1 | Generic application error |
| 125 | Docker failed to run container |
| 126 | Command found but not executable |
| 127 | Command not found |
| 128+N | Process terminated by signal N |
| 137 | Commonly SIGKILL; often OOM-related |

---

# 21. Advanced Docker Topics

## Q95. What is `docker commit`?

Creates a new image from a container's current filesystem state.

```bash
docker commit mycontainer myapp:snapshot
```

Useful for experimentation/debugging, but production images should normally be reproducibly built from Dockerfiles and source control.

---

## Q96. `docker commit` vs Dockerfile

| `docker commit` | Dockerfile |
|---|---|
| Captures current container state | Defines reproducible build |
| Manual | Declarative |
| Harder to audit | Easy to version-control |
| Not ideal for CI/CD | Excellent for CI/CD |

---

## Q97. What is `docker export`?

Exports a container filesystem as a tar archive.

```bash
docker export mycontainer -o container.tar
```

It is about the container filesystem, not a normal image backup with image history/metadata.

---

## Q98. `docker export` vs `docker save`

| `docker export` | `docker save` |
|---|---|
| Exports container filesystem | Exports image |
| Loses image layer history/metadata | Preserves image structure |
| Used for container filesystem export | Used for image transfer/backup |

---

## Q99. What does `docker diff` show?

Shows filesystem changes made inside a container compared with its image baseline.

```bash
docker diff <container>
```

Typical markers:

| Marker | Meaning |
|---|---|
| A | Added |
| C | Changed |
| D | Deleted |

---

## Q100. What is an init container?

An init container is a container used for initialization before the main application container starts. This concept is especially important in Kubernetes.

---

## Q101. What is a sidecar container?

A sidecar is a supporting container running alongside an application container, often providing functions such as logging, proxying, or configuration support.

Init and sidecar patterns are more directly implemented and managed by orchestration platforms such as Kubernetes.

---

# 22. Docker vs Kubernetes

## Q102. Is Docker the same as Kubernetes?

No.

Docker is primarily a container platform/runtime ecosystem. Kubernetes is a container orchestration platform.

Modern Kubernetes clusters commonly use CRI-compatible runtimes such as containerd or CRI-O rather than Docker Engine directly.

---

## Q103. Why do we need Kubernetes if Docker can run containers?

Docker Engine can run containers on individual hosts.

Kubernetes adds cluster-level capabilities such as:

- Scheduling
- Desired-state management
- Self-healing
- Service discovery
- Rolling updates
- Horizontal scaling
- Declarative configuration
- Cluster-wide orchestration

---

# 23. Scenario-Based Interview Questions

These questions should be practiced after learning the concepts above.

## Q104. Your container starts and immediately exits. Explain your debugging process.

### Answer framework

```text
1. docker ps -a
2. docker logs <container>
3. Check exit code
4. docker inspect <container>
5. Check CMD / ENTRYPOINT
6. Verify application process
7. Check environment/configuration
8. Run interactive shell if necessary
```

---

## Q105. Your application is running inside a container, but `localhost:8080` on the host cannot reach it.

Check:

```bash
docker ps
docker port <container>
docker inspect <container>
```

If the container listens on 8080 but no host port is published, run it with:

```bash
docker run -p 8080:8080 myapp
```

Also verify the application is listening on the expected interface and port.

---

## Q106. Two containers cannot communicate.

Answer:

```text
Check network membership
        ↓
docker network inspect
        ↓
Check DNS/service name
        ↓
Check application listening port
        ↓
Check container port
        ↓
Test from inside container
```

Example:

```bash
docker exec -it api sh
curl http://db:5432
```

For a database protocol, use the appropriate client/test rather than relying only on HTTP `curl`.

---

## Q107. A PostgreSQL container is recreated and all database data disappears. Why?

Likely the database data was stored only in the container writable layer.

Correct design:

```bash
docker volume create pgdata

docker run -d \
  -v pgdata:/var/lib/postgresql/data \
  postgres
```

---

## Q108. A Docker image is 2 GB. How would you reduce it?

Investigate:

```bash
docker history myapp:latest
docker image inspect myapp:latest
```

Then:

1. Use a suitable smaller base.
2. Use multistage builds.
3. Remove unnecessary packages.
4. Use `.dockerignore`.
5. Avoid copying build artifacts unnecessarily.
6. Clean package-manager caches where appropriate.
7. Copy only runtime artifacts.

---

## Q109. Docker builds are taking too long. What would you investigate?

Check:

```text
Build context size
       ↓
.dockerignore
       ↓
Dockerfile instruction order
       ↓
Cache invalidation
       ↓
Dependency installation
       ↓
Network/package registry latency
```

Example improvement:

```dockerfile
COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build
```

instead of copying the entire source before installing dependencies.

---

## Q110. Container exits with code 137.

Investigate:

```bash
docker inspect <container>
docker stats
dmesg | grep -i oom
journalctl -k | grep -i oom
```

Look for:

- Memory limit
- Memory usage
- Host memory pressure
- Kernel OOM events

Do not automatically conclude "Docker error" solely from 137.

---

## Q111. A container can resolve a hostname but cannot connect to the service.

Distinguish:

```text
DNS resolution
     ≠
TCP connectivity
     ≠
Application protocol
```

Test each layer independently.

---

## Q112. Your application works with `docker run` but fails under Compose.

Check:

- Environment variables
- Volume mounts
- Working directory
- Service names
- Network
- Port mappings
- Compose interpolation
- Startup/readiness assumptions
- Different image/build configuration

---

## Q113. A volume mount hides files that exist in the image.

Example:

```bash
docker run \
  -v /host/config:/app/config \
  myapp
```

If `/app/config` already contains image files, the mount can obscure those files while the mount is active.

This is a common container-storage troubleshooting question.

---

## Q114. A CI pipeline builds a Docker image but the deployment uses old code.

Possible causes:

```text
Wrong tag
   ↓
Registry push missing
   ↓
Deployment pulling cached image
   ↓
Mutable tag reused
   ↓
Incorrect registry/repository
```

Better production practice:

```text
Build
 ↓
Tag with immutable version/commit SHA
 ↓
Push
 ↓
Deploy exact image reference
```

Example:

```bash
docker build -t myapp:${GIT_COMMIT} .
docker push registry/myapp:${GIT_COMMIT}
```

---

## Q115. A container has access to the Docker socket. What security risk exists?

Access to the Docker daemon is highly privileged. A compromised process may be able to create containers with powerful host access.

Treat Docker socket access as equivalent to granting significant host-control capability.

---

## Q116. How would you design a production Docker CI/CD flow?

```text
Developer
   ↓
Git commit
   ↓
CI pipeline
   ↓
Unit tests
   ↓
Docker build
   ↓
Lint / security scan
   ↓
Tag with immutable version
   ↓
Push to registry
   ↓
Deployment
   ↓
Health verification
   ↓
Rollback if required
```

---

# 24. Rapid-Fire Questions

| Question | Short answer |
|---|---|
| What is Docker? | Containerization platform |
| Image vs container? | Template vs instance |
| VM vs container? | Full guest OS vs isolated process |
| Docker daemon? | `dockerd` |
| Docker runtime? | containerd + OCI runtime in common Docker architecture |
| Low-level OCI runtime? | `runc` is a common implementation |
| Namespaces? | Isolation |
| cgroups? | Resource control/accounting |
| `RUN`? | Build-time command |
| `CMD`? | Default runtime command/args |
| `ENTRYPOINT`? | Main runtime executable |
| `COPY`? | Copy files into image |
| `ADD`? | Copy plus additional features |
| `ARG`? | Build-time variable |
| `ENV`? | Runtime environment configuration |
| `.dockerignore`? | Excludes build-context files |
| `EXPOSE`? | Documents intended port |
| `-p`? | Publishes/maps host port |
| Volume? | Docker-managed persistent storage |
| Bind mount? | Host path mounted into container |
| `docker exec`? | New process in running container |
| `docker attach`? | Attach to existing main process |
| `docker logs`? | Container logs |
| `docker inspect`? | Detailed object metadata |
| `docker stats`? | Resource usage |
| Exit 137? | Commonly SIGKILL/OOM-related |
| Registry? | Image storage/distribution |
| ECR? | AWS container registry |
| Multistage build? | Separate build and runtime stages |
| Compose? | Declarative multi-container application |
| Swarm? | Docker-native orchestration |
| Kubernetes? | Container orchestration platform |

---

# 25. Important Command Cheat Sheet

## Images

```bash
docker images
docker pull nginx
docker build -t myapp:1.0 .
docker tag myapp:1.0 registry/myapp:1.0
docker push registry/myapp:1.0
docker rmi myapp:1.0
docker history myapp:1.0
docker image inspect myapp:1.0
```

## Containers

```bash
docker run nginx
docker run -d --name web nginx
docker ps
docker ps -a
docker start web
docker stop web
docker restart web
docker kill web
docker rm web
docker exec -it web sh
docker logs web
docker logs -f web
docker inspect web
docker stats web
docker top web
docker diff web
```

## Networking

```bash
docker network ls
docker network create app-net
docker network inspect app-net
docker network connect app-net web
docker network disconnect app-net web
docker port web
```

## Storage

```bash
docker volume ls
docker volume create appdata
docker volume inspect appdata
docker volume rm appdata
```

## Cleanup

```bash
docker system df
docker system df -v
docker container prune
docker image prune
docker network prune
docker volume prune
docker system prune
```

---

# 26. Common Interview Traps

## Trap 1: "EXPOSE publishes the port."

**Wrong.**

`EXPOSE` documents the intended port.

```dockerfile
EXPOSE 8080
```

does not publish it.

Use:

```bash
docker run -p 8080:8080 myapp
```

---

## Trap 2: "Container has its own kernel."

**Wrong for normal Linux containers.**

Containers share the host kernel.

---

## Trap 3: "Docker image is the running application."

**Wrong.**

Image = template.

Container = runtime instance.

---

## Trap 4: "CMD always runs exactly as written."

**Wrong.**

CMD can be overridden by the command supplied to `docker run`.

---

## Trap 5: "`docker stop` and `docker kill` are identical."

**Wrong.**

`stop` attempts graceful termination; `kill` sends a kill signal by default.

---

## Trap 6: "localhost means the Docker host."

**Wrong.**

Inside a container, localhost normally means that container's own network namespace.

---

## Trap 7: "A volume disappears when the container is removed."

**Usually wrong for named volumes.**

Named volumes persist until explicitly removed.

---

## Trap 8: "Exit 137 always means OOM."

**Too absolute.**

137 means the process exited due to signal 9 (`SIGKILL`) in the conventional shell encoding. OOM is a common cause, but verify.

---

## Trap 9: "Docker Compose `depends_on` means the dependency is ready."

**Wrong.**

Startup order is not the same as application readiness.

---

## Trap 10: "Docker is Kubernetes."

**Wrong.**

Docker and Kubernetes solve different layers/problems, although they operate in the same container ecosystem.

---

# 27. Final Interview Checklist

Before an interview, make sure you can explain all of these **without memorizing definitions only**.

### Fundamentals

- [ ] What is Docker?
- [ ] What is a container?
- [ ] Container vs VM
- [ ] Image vs container
- [ ] Why containers are lightweight
- [ ] Container lifecycle

### Architecture

- [ ] Docker Client
- [ ] Docker Daemon
- [ ] Docker API/socket
- [ ] containerd
- [ ] runc
- [ ] OCI
- [ ] Namespaces
- [ ] cgroups
- [ ] PID 1

### Images

- [ ] Image layers
- [ ] Build cache
- [ ] Tags
- [ ] Digests
- [ ] Image history
- [ ] Image inspection

### Dockerfile

- [ ] FROM
- [ ] RUN
- [ ] COPY
- [ ] ADD
- [ ] CMD
- [ ] ENTRYPOINT
- [ ] ENV
- [ ] ARG
- [ ] WORKDIR
- [ ] USER
- [ ] EXPOSE
- [ ] `.dockerignore`

### Networking

- [ ] Bridge
- [ ] Host
- [ ] None
- [ ] Overlay
- [ ] Port publishing
- [ ] EXPOSE vs `-p`
- [ ] Container localhost
- [ ] Container-to-container communication
- [ ] Docker DNS

### Storage

- [ ] Writable container layer
- [ ] Volumes
- [ ] Bind mounts
- [ ] Persistence
- [ ] Mount troubleshooting

### Production

- [ ] Multistage builds
- [ ] Image optimization
- [ ] Non-root containers
- [ ] Image scanning
- [ ] Registry
- [ ] ECR
- [ ] Immutable image tags/digests
- [ ] Secret handling
- [ ] Resource limits

### Troubleshooting

- [ ] Container exits
- [ ] Crash/restart loop
- [ ] Exit codes
- [ ] Port problems
- [ ] DNS problems
- [ ] Network connectivity
- [ ] Permission problems
- [ ] Volume problems
- [ ] Docker daemon problems
- [ ] Disk exhaustion
- [ ] OOM

### Orchestration

- [ ] Compose
- [ ] Swarm
- [ ] Kubernetes
- [ ] Docker vs Kubernetes
- [ ] Runtime vs orchestrator

---

# Interview Answer Formula

For scenario questions, avoid giving only a command.

Use this structure:

```text
1. Identify the symptom
2. Explain the likely causes
3. Show the first diagnostic command
4. Explain what the output tells you
5. Narrow down the problem
6. Apply the fix
7. Explain how you would prevent recurrence
```

### Example

**Question:** Container is continuously restarting.

Weak answer:

```bash
docker restart container
```

Better answer:

```text
First I would inspect the container state and logs:

docker ps -a
docker logs --tail 200 <container>
docker inspect <container>

Then I would check the exit code and determine whether the
application is crashing, the command is incorrect, configuration
is missing, the health check is failing, or the container is being
killed because of resource pressure.

After identifying the root cause, I would fix the application,
configuration, image or resource settings rather than repeatedly
restarting the container.
```

---

# Final Preparation Strategy

Study Docker in this sequence:

```text
Docker Fundamentals
        ↓
Docker Architecture
        ↓
Namespaces + cgroups
        ↓
Images + Layers
        ↓
Dockerfile
        ↓
RUN/CMD/ENTRYPOINT
        ↓
Multistage Builds
        ↓
Container Commands
        ↓
Networking
        ↓
Storage
        ↓
Registries
        ↓
Resources
        ↓
Compose
        ↓
Security
        ↓
Troubleshooting
        ↓
Advanced Concepts
        ↓
Docker vs Kubernetes
        ↓
Scenario-Based Questions
```

The most important goal is not memorizing all commands. You should be able to explain **what happens internally, why a configuration works, how it fails, and how you would troubleshoot it**.
