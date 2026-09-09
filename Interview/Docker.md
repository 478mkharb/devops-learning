# Docker Interview Preparation

A structured Docker interview-preparation guide covering **fundamentals, architecture, internals, images, Dockerfiles, builds, commands, networking, storage, configuration, registries, resources, lifecycle, troubleshooting, exit codes, orchestration, advanced concepts, scenarios, and Docker/Kubernetes comparison**.

The questions are ordered from foundational concepts to practical production troubleshooting.

## Topic Structure

```text
Docker Interview Preparation
│
├── 1. Docker Fundamentals
├── 2. Docker Architecture
├── 3. Docker Internals
├── 4. Images & Layers
├── 5. Dockerfile
├── 6. Multistage Builds
├── 7. Docker Commands
├── 8. Docker Networking
├── 9. Docker Storage
├── 10. Environment Variables
├── 11. Container Registries
├── 12. Resource Management
├── 13. Container Lifecycle
├── 14. Troubleshooting
├── 15. Exit Codes
├── 16. Scaling & Orchestration
├── 17. Advanced Docker
├── 18. Scenario-Based Questions
├── 19. Docker vs Kubernetes
├── 20. Rapid-Fire Questions
├── 21. Common Interview Traps
├── 22. Important Commands Cheat Sheet
└── 23. Final Interview Checklist
```

---

# 1. Docker Fundamentals

## Q1. What is Docker?

**Definition:** Docker is a platform for building, packaging, distributing, and running applications as containers.

Docker packages an application with the files and dependencies required for its runtime environment into an image.

```text
Application + Dependencies
          ↓
      Docker Image
          ↓
      Container
          ↓
    Application Process
```

---

## Q2. What is a container?

**Definition:** A container is an isolated process created from a container image.

A Linux container normally shares the host Linux kernel while using isolation mechanisms such as namespaces and resource controls such as cgroups.

---

## Q3. Container vs Virtual Machine

| Feature | Container | Virtual Machine |
|---|---|---|
| Main abstraction | Isolated processes | Virtual machine/guest OS |
| Kernel | Normally shares host kernel | Guest OS has its own kernel |
| Startup | Usually fast | Usually slower |
| Resource overhead | Generally lower | Generally higher |
| Typical use | Microservices, CI/CD | Full OS workloads |

**Interview point:** A container is not simply a "small VM."

---

## Q4. Why are containers lightweight?

Containers normally do not boot a complete guest operating system. They share the host kernel and package the application environment rather than a full guest OS.

That usually means lower startup and memory overhead than a VM, although containers still consume CPU, memory, storage, and network resources.

---

## Q5. Image vs Container

| Image | Container |
|---|---|
| Immutable package/template | Runtime instance |
| Contains filesystem and metadata | Uses image plus a writable container layer |
| Used to create containers | Created from an image |
| Can be stored in a registry | Runtime object on a host |

Example:

```bash
docker pull nginx
docker run -d --name web nginx
```

---

## Q6. Why is Docker useful in CI/CD?

A build can produce a versioned container image that can be promoted through environments.

```text
Git commit
   ↓
CI build
   ↓
Docker image
   ↓
Security scan
   ↓
Registry
   ↓
Deployment
```

This reduces environment differences and improves artifact traceability.

---

# 2. Docker Architecture

## Q7. Explain Docker architecture.

A simplified Linux Docker Engine flow is:

```text
Docker CLI
    ↓
Docker Engine API
    ↓
dockerd
    ↓
containerd
    ↓
OCI runtime (commonly runc)
    ↓
Linux kernel
    ├── namespaces
    └── cgroups
```

The exact implementation can vary by Docker release and platform, but this is a useful interview mental model.

---

## Q8. What is the Docker Client?

The Docker Client, commonly the `docker` CLI, is the user-facing interface used to send requests to Docker Engine.

Examples:

```bash
docker ps
docker build -t myapp:1.0 .
docker run nginx
```

---

## Q9. What is `dockerd`?

`dockerd` is the Docker Engine daemon. It receives Docker API requests and manages Docker objects such as:

- Containers
- Images
- Networks
- Volumes

---

## Q10. What is the Docker API?

The Docker Engine API is the interface used by clients and tools to request operations from Docker Engine.

Conceptually:

```text
docker CLI
    ↓
Docker API
    ↓
dockerd
```

---

## Q11. What is `/var/run/docker.sock`?

On a typical Linux installation, the Docker daemon exposes a Unix socket at:

```text
/var/run/docker.sock
```

The Docker CLI commonly uses this local socket to communicate with the daemon.

**Security:** Access to the Docker daemon is highly privileged. Giving an untrusted process access to the socket can provide extensive control over the host.

---

## Q12. What happens when you run `docker run nginx`?

A simplified flow is:

```text
docker run nginx
      ↓
Docker CLI
      ↓
Docker API
      ↓
dockerd
      ↓
Check local image
      ↓
Pull image if necessary
      ↓
Create container configuration
      ↓
Configure filesystem/network
      ↓
Container runtime creates process
      ↓
nginx process starts
```

---

# 3. Docker Internals

## Q13. What are Linux namespaces?

**Definition:** Linux namespaces isolate system resources so that a process has a restricted view of the system.

| Namespace | Main isolation |
|---|---|
| PID | Process IDs |
| NET | Network interfaces/routes |
| MNT | Mount points/filesystem view |
| UTS | Hostname/domain name |
| IPC | IPC resources |
| USER | User/group IDs |

---

## Q14. What are cgroups?

**Definition:** Control groups are Linux kernel mechanisms for accounting for and controlling resource usage by groups of processes.

Typical resources include:

- CPU
- Memory
- PIDs
- I/O

Example:

```bash
docker run --cpus=1 --memory=512m nginx
```

---

## Q15. Namespace vs cgroup

| Namespace | cgroup |
|---|---|
| Isolation | Resource control/accounting |
| "What can the process see?" | "How much can it use?" |
| PID/network/filesystem examples | CPU/memory examples |

---

## Q16. What is containerd?

`containerd` is a container runtime component responsible for container lifecycle management and related image/runtime operations. Docker Engine uses containerd as part of its runtime architecture.

---

## Q17. What is runc?

`runc` is a low-level OCI-compliant container runtime implementation used to create and run containers according to OCI runtime specifications.

---

## Q18. What is OCI?

**OCI = Open Container Initiative.**

OCI publishes open standards for container image formats and container runtimes.

The two concepts commonly discussed in interviews are:

- OCI Image Specification
- OCI Runtime Specification

---

## Q19. What is PID 1 inside a container?

The first process in a container's PID namespace normally has PID 1.

PID 1 has important process-management responsibilities, particularly around signal handling and reaping orphaned child processes.

This is one reason the application process and shutdown behavior matter in containers.

---

# 4. Images & Layers

## Q20. What is a Docker image?

**Definition:** A Docker image is an immutable, layered package containing filesystem content and metadata used to create containers.

Example:

```text
myapp:1.0
```

An image itself is not a running process.

---

## Q21. What is a Docker image layer?

A layer represents filesystem changes that form part of an image.

Example:

```dockerfile
FROM ubuntu:24.04
RUN apt-get update && apt-get install -y nginx
COPY index.html /var/www/html/
```

Conceptually:

```text
Base layer
    +
Package layer
    +
Application layer
    =
Final image
```

---

## Q22. Why are Docker images layered?

Layers allow:

- Reuse of common content
- Build-cache reuse
- Efficient distribution of unchanged content
- Incremental image changes

---

## Q23. What is the Docker build cache?

Docker can reuse previously built results when the relevant instruction and inputs have not changed.

Good cache-oriented ordering:

```dockerfile
COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build
```

A source-code change can then avoid invalidating the dependency-installation step.

---

## Q24. What is a Docker image tag?

A tag is a human-readable reference such as:

```text
myapp:1.4.0
myapp:latest
```

Tags are mutable references; the same tag can later point to different image content.

---

## Q25. What is an image digest?

A digest is a content-addressed identifier for image content.

Example:

```text
myapp@sha256:<digest>
```

A digest is useful when an exact immutable image reference is required.

---

## Q26. Tag vs Digest

| Tag | Digest |
|---|---|
| Human-readable | Content-addressed |
| Can move to different content | Identifies exact content |
| Convenient for development | Useful for immutable deployment references |

---

## Q27. What is copy-on-write in Docker?

A container normally uses read-only image layers plus a writable container layer.

```text
Image layer 3
Image layer 2
Image layer 1
----------------
Writable layer
----------------
Container filesystem
```

Changes made by the container are stored in the writable layer rather than modifying the original image layers.

---

# 5. Dockerfile

## Q28. What is a Dockerfile?

**Definition:** A Dockerfile is a text file containing instructions used to build a container image.

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
docker build -t myapp:1.0 .
```

---

## Q29. What does `FROM` do?

`FROM` selects the base image and starts a build stage.

```dockerfile
FROM python:3.12-slim
```

A multistage Dockerfile can contain multiple `FROM` instructions.

---

## Q30. What does `RUN` do?

`RUN` executes a command **during image build time**.

```dockerfile
RUN apt-get update && apt-get install -y curl
```

The resulting filesystem changes become part of the image.

**Do not confuse it with `CMD`:**

```text
RUN → build time
CMD → container runtime
```

---

## Q31. What does `CMD` do?

**Definition:** `CMD` specifies the default command and/or default arguments used when a container starts.

```dockerfile
CMD ["python", "app.py"]
```

A command supplied to:

```bash
docker run myapp <command>
```

normally replaces the image's default `CMD`.

---

## Q32. What does `ENTRYPOINT` do?

**Definition:** `ENTRYPOINT` specifies the primary executable that the container is intended to run.

Example:

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

Then:

```bash
docker run myapp
```

conceptually runs:

```text
python app.py
```

---

## Q33. `CMD` vs `ENTRYPOINT`

| `CMD` | `ENTRYPOINT` |
|---|---|
| Default command/arguments | Primary executable |
| Easy to replace with a runtime command | Intended to define the main executable |
| Commonly supplies default arguments | Commonly paired with `CMD` |

Example:

```dockerfile
ENTRYPOINT ["sleep"]
CMD ["1000"]
```

```bash
docker run myapp 500
```

conceptually runs:

```text
sleep 500
```

---

## Q34. What is shell form vs exec form?

Shell form:

```dockerfile
CMD python app.py
```

Exec form:

```dockerfile
CMD ["python", "app.py"]
```

Exec form is generally preferred for application processes because it avoids an extra shell and usually provides cleaner signal handling.

---

## Q35. What does `COPY` do?

`COPY` copies files/directories from the build context into the image, or from another build stage.

```dockerfile
COPY app.py /app/
```

Multistage example:

```dockerfile
COPY --from=builder /src/app /usr/local/bin/app
```

---

## Q36. What does `ADD` do?

`ADD` copies files/directories and also supports additional Docker-specific behavior, including extraction of local tar archives.

Example:

```dockerfile
ADD app.tar.gz /app/
```

For ordinary file copying, `COPY` is generally clearer and preferred.

---

## Q37. `COPY` vs `ADD`

| `COPY` | `ADD` |
|---|---|
| Straightforward file/directory copying | Copy plus additional behavior |
| Explicit and simple | More feature-rich |
| Preferred for ordinary copying | Use when ADD-specific behavior is actually needed |

---

## Q38. What does `ARG` do?

`ARG` defines a build-time variable.

```dockerfile
ARG APP_VERSION=1.0
RUN echo "Building $APP_VERSION"
```

Build:

```bash
docker build --build-arg APP_VERSION=2.0 -t myapp:2.0 .
```

`ARG` should not be treated as a secure secret store.

---

## Q39. What does `ENV` do?

`ENV` defines environment variables in the image.

```dockerfile
ENV APP_ENV=production
```

A runtime value can override it:

```bash
docker run -e APP_ENV=staging myapp
```

---

## Q40. `ARG` vs `ENV`

| `ARG` | `ENV` |
|---|---|
| Build-time variable | Container environment variable |
| Set with `--build-arg` | Set/override with `-e` |
| Primarily build configuration | Runtime/application configuration |
| Not automatically a runtime environment variable | Available in the container environment |

Neither should be used as a general-purpose secret-management mechanism.

---

## Q41. What does `WORKDIR` do?

`WORKDIR` sets the working directory for subsequent Dockerfile instructions and the default working directory for the container.

```dockerfile
WORKDIR /app
COPY . .
```

---

## Q42. What does `USER` do?

`USER` specifies the user/group used by subsequent Dockerfile instructions and by default for the container process.

```dockerfile
USER 10001
```

Running as a non-root user is generally preferred when practical.

---

## Q43. What does `EXPOSE` do?

`EXPOSE` documents the port that the application intends to listen on inside the container.

```dockerfile
EXPOSE 8080
```

It **does not publish** the port to the host.

Publishing is done at runtime, for example:

```bash
docker run -p 8080:8080 myapp
```

---

## Q44. What does `VOLUME` do?

`VOLUME` declares a mount point intended for externalized/persistent data.

```dockerfile
VOLUME ["/var/lib/myapp"]
```

In many deployments, volumes are explicitly managed at runtime or by an orchestrator.

---

## Q45. What does `LABEL` do?

`LABEL` adds metadata to an image.

```dockerfile
LABEL version="1.0"
LABEL team="platform"
```

Inspect:

```bash
docker image inspect myapp:1.0
```

---

## Q46. What does `HEALTHCHECK` do?

`HEALTHCHECK` defines a test Docker can use to determine whether a container is healthy.

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:8080/health || exit 1
```

A health status is different from merely knowing that the main process exists.

---

## Q47. What do `SHELL`, `STOPSIGNAL`, and `ONBUILD` do?

| Instruction | Definition |
|---|---|
| `SHELL` | Changes the default shell used by shell-form instructions |
| `STOPSIGNAL` | Defines the signal Docker uses to stop the container |
| `ONBUILD` | Registers a deferred instruction that runs when the image is later used as a base |

Examples:

```dockerfile
SHELL ["/bin/bash", "-c"]
STOPSIGNAL SIGTERM
ONBUILD COPY . /app
```

---

## Q48. What is `.dockerignore`?

`.dockerignore` excludes files/directories from the Docker build context.

Example:

```text
.git
node_modules
__pycache__
*.log
.env
terraform.tfstate
```

It can reduce build-context size and help prevent unnecessary or sensitive files from being sent as build context.

---

## Q49. What is Docker build context?

In:

```bash
docker build -t myapp:1.0 .
```

the `.` is the build context.

The build context is the set of files made available to the build according to Docker's build rules. `.dockerignore` can exclude files from it.

---

# 6. Multistage Builds

## Q50. What is a multistage build?

A multistage Dockerfile uses multiple `FROM` stages so build dependencies can be separated from the final runtime image.

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

---

## Q51. Why use multistage builds?

| Without multistage | With multistage |
|---|---|
| Compiler may remain in final image | Compiler excluded |
| Larger image | Smaller runtime image |
| More runtime packages | Only required runtime content |
| Larger attack surface | Reduced runtime surface |

---

## Q52. How do you optimize Docker images?

Common practices:

1. Choose an appropriate base image.
2. Use multistage builds.
3. Use `.dockerignore`.
4. Optimize instruction ordering for cache reuse.
5. Install only required packages.
6. Avoid unnecessary files.
7. Remove package-manager caches where appropriate.
8. Run as non-root where practical.
9. Scan images for vulnerabilities.
10. Use traceable/immutable image references for production.

---

# 7. Docker Commands

## Q53. How do you list containers?

```bash
docker ps
```

All containers:

```bash
docker ps -a
```

---

## Q54. How do you list and inspect images?

```bash
docker images
docker image inspect myapp:1.0
docker history myapp:1.0
```

---

## Q55. How do you build an image?

```bash
docker build -t myapp:1.0 .
```

With a build argument:

```bash
docker build \
  --build-arg APP_VERSION=2.0 \
  -t myapp:2.0 .
```

---

## Q56. How do you pull, tag, and push an image?

```bash
docker pull nginx:1.27

docker tag myapp:1.0 registry.example.com/team/myapp:1.0

docker push registry.example.com/team/myapp:1.0
```

---

## Q57. How do you inspect a container?

```bash
docker inspect web
```

Useful information includes:

- State
- IP address
- Mounts
- Environment
- Network configuration
- Entrypoint
- Command
- Resource configuration

---

## Q58. How do you view container logs?

```bash
docker logs web
docker logs -f web
docker logs --tail 100 web
```

---

## Q59. How do you execute a command inside a running container?

```bash
docker exec -it web sh
```

or:

```bash
docker exec -it web bash
```

when Bash exists in the image.

---

## Q60. `docker exec` vs `docker attach`

| `docker exec` | `docker attach` |
|---|---|
| Starts a new process | Attaches to existing main process I/O |
| Good for troubleshooting | Useful for interacting with the main process |
| Example: `docker exec -it web sh` | Example: `docker attach web` |

---

## Q61. How do you inspect filesystem changes?

```bash
docker diff web
```

Typical markers:

| Marker | Meaning |
|---|---|
| `A` | Added |
| `C` | Changed |
| `D` | Deleted |

---

## Q62. How do you inspect Docker disk usage?

```bash
docker system df
docker system df -v
```

Then inspect images, containers, and volumes before deleting anything.

---

# 8. Docker Networking

## Q63. What are common Docker network drivers?

| Driver | Purpose |
|---|---|
| `bridge` | Common single-host container networking |
| `host` | Uses host network namespace |
| `none` | No normal network connectivity |
| `overlay` | Multi-host networking, commonly used with orchestration |

---

## Q64. What is a user-defined bridge network?

It is a Docker bridge network created by the user for controlled container connectivity.

```bash
docker network create app-net

docker run -d --name db --network app-net postgres
docker run -d --name api --network app-net my-api
```

Containers on the same user-defined network can communicate using Docker's embedded DNS/service-name mechanisms.

---

## Q65. What is host networking?

With:

```bash
docker run --network host myapp
```

the container uses the host's network namespace rather than the normal isolated container network namespace.

This reduces network isolation and should be used deliberately.

---

## Q66. What is `none` networking?

```bash
docker run --network none alpine
```

This disables normal container network connectivity.

---

## Q67. What is port publishing?

```bash
docker run -d -p 8080:80 nginx
```

means:

```text
Host TCP 8080
      ↓
Container TCP 80
```

---

## Q68. `EXPOSE` vs `-p`

| `EXPOSE` | `-p` |
|---|---|
| Dockerfile metadata/documentation | Runtime port publishing |
| Does not publish host port | Publishes/maps host port |
| `EXPOSE 80` | `-p 8080:80` |

---

## Q69. Why does `localhost` cause Docker networking problems?

`localhost` refers to the current network namespace.

Inside an API container:

```text
localhost:5432
```

means port 5432 in the API container, not the database container.

For containers on a shared user-defined network, use the database service/container name:

```text
db:5432
```

---

## Q70. How do you troubleshoot Docker networking?

Start with:

```bash
docker ps
docker port <container>
docker network ls
docker network inspect <network>
docker inspect <container>
```

Then enter the container:

```bash
docker exec -it <container> sh
```

Test DNS:

```bash
getent hosts db
```

Test TCP:

```bash
nc -vz db 5432
```

Test HTTP:

```bash
curl http://api:8080
```

Separate the investigation into:

```text
DNS → TCP connectivity → Application protocol
```

---

# 9. Docker Storage

## Q71. Why is the container writable layer considered ephemeral?

A container has a writable layer above its read-only image layers.

When the container is removed, that writable layer is normally removed.

Therefore, important application data should be externalized.

---

## Q72. What is a Docker volume?

**Definition:** A Docker volume is Docker-managed persistent storage.

```bash
docker volume create pgdata

docker run -d \
  --name postgres \
  -v pgdata:/var/lib/postgresql/data \
  postgres
```

---

## Q73. What is a bind mount?

A bind mount maps an explicit host filesystem path into the container.

```bash
docker run -d \
  -v /opt/myapp/config:/app/config \
  myapp
```

---

## Q74. Volume vs bind mount

| Volume | Bind mount |
|---|---|
| Managed by Docker | Explicit host path |
| Less coupled to host path layout | Strong host filesystem dependency |
| Good for persistent application data | Useful when host path control is required |
| Docker manages storage location | User selects source path |

---

## Q75. Does removing a container remove a named volume?

Normally, no.

```bash
docker rm postgres
```

does not normally delete:

```text
pgdata
```

The volume must be removed separately:

```bash
docker volume rm pgdata
```

---

# 10. Environment Variables

## Q76. How do you pass environment variables to a container?

```bash
docker run \
  -e APP_ENV=production \
  -e DB_HOST=db \
  myapp
```

---

## Q77. How can runtime configuration override an image `ENV`?

Dockerfile:

```dockerfile
ENV APP_ENV=production
```

Runtime:

```bash
docker run -e APP_ENV=staging myapp
```

The runtime value overrides the image default.

---

## Q78. Should Docker `ARG` or `ENV` be used for secrets?

Not as a general secret-management mechanism.

Avoid embedding credentials into an image through:

```dockerfile
ENV DB_PASSWORD=secret
```

or treating build arguments as a secure secret store.

Use an appropriate secret-management solution for the build/deployment platform.

---

# 11. Container Registries

## Q79. What is a container registry?

A container registry stores and distributes container images.

Examples include:

- Docker Hub
- Amazon ECR
- GitHub Container Registry
- Private enterprise registries

---

## Q80. What is Amazon ECR?

Amazon Elastic Container Registry (ECR) is AWS's managed container registry.

A common CI/CD flow is:

```text
Source code
   ↓
CI build
   ↓
docker build
   ↓
Security scan
   ↓
docker push
   ↓
ECR
   ↓
Deployment
```

---

## Q81. Why should production images be traceable?

A deployment should allow you to answer:

```text
Which source commit produced this image?
Which image was deployed?
Which registry artifact was used?
```

A useful pattern is:

```text
myapp:<git-commit-sha>
```

and/or an immutable digest.

---

# 12. Resource Management

## Q82. How do you limit container memory?

```bash
docker run --memory=512m myapp
```

---

## Q83. How do you limit CPU?

```bash
docker run --cpus=1 myapp
```

This constrains CPU usage; it does not mean the container owns one dedicated physical CPU core.

---

## Q84. How do you monitor container resource usage?

```bash
docker stats
```

Specific container:

```bash
docker stats myapp
```

---

## Q85. What can cause an OOM-related container termination?

Possible causes include:

- Container memory limit being exceeded
- Host memory pressure
- A process being killed by the kernel

A common symptom is exit code 137, but **137 alone does not prove OOM**.

Verify with:

```bash
docker inspect <container>
dmesg | grep -i oom
journalctl -k | grep -i oom
```

---

## Q86. How do you clean unused Docker resources safely?

Inspect first:

```bash
docker system df -v
```

Then selectively prune:

```bash
docker container prune
docker image prune
docker network prune
docker volume prune
```

Be particularly careful with:

```bash
docker system prune -a --volumes
```

because it can remove unused images, containers, networks, and volumes.

---

# 13. Container Lifecycle

## Q87. Explain the container lifecycle.

```text
Image
  ↓
create
  ↓
Created
  ↓
start
  ↓
Running
  ↓
stop
  ↓
Stopped/Exited
  ↓
rm
  ↓
Removed
```

`docker run` normally combines creation and startup.

---

## Q88. `docker run` vs `docker start`

| Command | Meaning |
|---|---|
| `docker run IMAGE` | Creates a new container and starts it |
| `docker start CONTAINER` | Starts an existing container |

---

## Q89. `docker stop` vs `docker kill`

| Command | Behavior |
|---|---|
| `docker stop` | Attempts graceful termination, then forcefully terminates after timeout |
| `docker kill` | Sends a kill signal immediately by default |

---

## Q90. Why does a container exit when the main process exits?

A container's lifecycle is normally tied to its primary process.

```text
Main process starts
       ↓
Application runs
       ↓
Main process exits
       ↓
Container exits
```

A restart policy or orchestrator can restart it if configured.

---

## Q91. What are Docker restart policies?

Examples:

```bash
docker run --restart=no myapp
docker run --restart=on-failure myapp
docker run --restart=always myapp
docker run --restart=unless-stopped myapp
```

Restart policies control automatic restart behavior; they do not fix the underlying application failure.

---

# 14. Troubleshooting

## Q92. What is your general Docker troubleshooting approach?

Use a structured sequence:

```text
Container state
     ↓
Exit code
     ↓
Logs
     ↓
Inspect configuration
     ↓
CMD/ENTRYPOINT
     ↓
Environment/config files
     ↓
Networking
     ↓
Storage/permissions
     ↓
CPU/memory
     ↓
Docker daemon/host
```

---

## Q93. Container exits immediately. What do you check?

```bash
docker ps -a
docker logs <container>
docker inspect <container>
```

Then check:

- Exit code
- `CMD`
- `ENTRYPOINT`
- Environment variables
- Configuration files
- Dependencies
- Permissions
- Application startup behavior

---

## Q94. Container is continuously restarting. What do you check?

```bash
docker ps
docker logs --tail 200 <container>
docker inspect <container>
```

Investigate:

- Application crash
- Wrong command
- Missing configuration
- Dependency failure
- Healthcheck
- Memory/resource limits
- Restart policy

---

## Q95. Application works on the host but not inside Docker. What do you investigate?

Check:

```text
Application bind address
        ↓
Container port
        ↓
Published host port
        ↓
Docker network
        ↓
Firewall
        ↓
Application configuration
```

A common issue is an application listening only on `127.0.0.1` inside the container when it needs to accept connections through the container network.

---

## Q96. Docker container cannot reach another container. What do you check?

```bash
docker network inspect <network>
docker inspect <api>
docker inspect <db>
```

Then:

```bash
docker exec -it api sh
getent hosts db
nc -vz db 5432
```

Determine whether the failure is:

```text
DNS
 ↓
TCP
 ↓
Application protocol
```

---

## Q97. Container cannot access the internet. What do you check?

Inside the container:

```bash
ip addr
ip route
cat /etc/resolv.conf
```

Then investigate:

- DNS
- Default route
- Docker network
- Host connectivity
- NAT
- Firewall
- Proxy configuration

---

## Q98. Port mapping is not working. What do you check?

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

Verify that:

```text
Host 8080 → Container 80
```

and that the application is actually listening on the expected container port/interface.

---

## Q99. You get "permission denied" inside a container. What do you investigate?

```bash
docker exec -it <container> sh
id
ls -la
```

Consider:

- Container user
- File ownership
- Bind-mount permissions
- Read-only mounts
- Host filesystem permissions

---

## Q100. Docker works with `sudo` but not without it. What do you check?

```bash
ls -l /var/run/docker.sock
groups
docker context ls
```

The user may not have the required access to the Docker daemon/socket.

Remember that Docker daemon access is highly privileged.

---

## Q101. Docker host is running out of disk space. How do you troubleshoot?

```bash
df -h
docker system df -v
```

Then inspect:

```bash
docker images
docker ps -a
docker volume ls
```

Remove unused resources selectively after confirming they are safe to delete.

---

# 15. Exit Codes

## Q102. What does exit code 0 mean?

The process completed successfully.

---

## Q103. What does exit code 1 mean?

A generic application-level failure. The exact meaning depends on the application.

---

## Q104. What do exit codes 125, 126, and 127 commonly mean?

| Code | Common meaning |
|---:|---|
| 125 | Docker failed to run the container command |
| 126 | Command was found but could not be executed |
| 127 | Command was not found |

These are conventions/common interpretations; always inspect logs and command configuration.

---

## Q105. What does exit code 137 mean?

137 commonly corresponds to:

```text
128 + 9 = 137
```

Signal 9 is `SIGKILL`.

OOM killing is a common cause, but 137 does **not** prove OOM.

Verify kernel/container evidence.

---

# 16. Scaling & Orchestration

## Q106. Can Docker run multiple instances of an application?

Yes.

```bash
docker run -d --name api1 myapp
docker run -d --name api2 myapp
docker run -d --name api3 myapp
```

Managing many instances reliably across hosts requires orchestration.

---

## Q107. What is Docker Compose?

Docker Compose is a declarative tool for defining and running multi-container applications.

Example:

```yaml
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: example

  api:
    build: .
    ports:
      - "8080:8080"
    depends_on:
      - db
```

---

## Q108. Does Compose `depends_on` guarantee application readiness?

No.

It can express dependency/startup ordering, but a dependency can be started before its application is actually ready to accept requests.

Use appropriate healthchecks and application retry/readiness logic.

---

## Q109. What is Docker Swarm?

Docker Swarm is Docker's native container orchestration technology for managing services across multiple nodes.

It provides concepts such as:

- Managers
- Workers
- Services
- Tasks
- Desired state
- Scheduling
- Scaling

---

## Q110. What is the difference between Docker Engine and an orchestrator?

| Docker Engine/runtime platform | Orchestrator |
|---|---|
| Runs/manages containers on a host | Manages workloads across a cluster |
| Container/image/network/volume operations | Scheduling, desired state, scaling, rollout |
| Example: Docker Engine | Examples: Kubernetes, Swarm |

---

# 17. Advanced Docker

## Q111. What is `docker commit`?

`docker commit` creates a new image from a container's current filesystem state.

```bash
docker commit mycontainer myapp:snapshot
```

It can be useful for experimentation, but Dockerfiles are generally preferred for reproducible builds.

---

## Q112. `docker commit` vs Dockerfile

| `docker commit` | Dockerfile |
|---|---|
| Captures current container state | Defines reproducible build instructions |
| Manual | Version-controllable |
| Harder to reproduce | Better for CI/CD |
| Useful for experiments | Preferred production approach |

---

## Q113. `docker export` vs `docker save`

| `docker export` | `docker save` |
|---|---|
| Exports a container filesystem | Exports a Docker image |
| Container-oriented | Image-oriented |
| Does not preserve the normal image layer/history structure | Preserves image layers/metadata |

Examples:

```bash
docker export mycontainer -o container.tar
docker save -o myapp.tar myapp:1.0
```

---

## Q114. What is the difference between a container runtime and an orchestrator?

A runtime creates/runs containers.

An orchestrator manages workloads across one or more nodes.

```text
Orchestrator
    ↓
Runtime
    ↓
Container
    ↓
Process
```

Examples:

```text
Kubernetes / Swarm → orchestration
containerd / runc  → runtime components
```

---

## Q115. Why is mounting `/var/run/docker.sock` into a container security-sensitive?

Example:

```bash
-v /var/run/docker.sock:/var/run/docker.sock
```

This gives the container access to the host Docker daemon.

A compromised workload may therefore gain extensive control over Docker-managed resources and potentially the host.

Treat Docker daemon access as a privileged security boundary.

---

# 18. Scenario-Based Questions

## Q116. Scenario: Container exits immediately

You start:

```bash
docker run myapp
```

and it exits.

### Answer approach

```bash
docker ps -a
docker logs myapp
docker inspect myapp
```

Check:

```text
Exit code
CMD
ENTRYPOINT
Environment
Configuration
Dependencies
Permissions
```

Find the root cause rather than repeatedly restarting it.

---

## Q117. Scenario: Application is inaccessible from the host

The application listens on container port 8080.

You run:

```bash
docker run -d myapp
```

but:

```bash
curl http://localhost:8080
```

fails.

### Likely issue

The container port was not published.

Use:

```bash
docker run -d -p 8080:8080 myapp
```

Also verify that the application is listening on the expected interface and port.

---

## Q118. Scenario: API cannot connect to PostgreSQL

The API is configured with:

```text
DB_HOST=localhost
DB_PORT=5432
```

PostgreSQL runs in another container.

### Root cause

Inside the API container:

```text
localhost = API container
```

### Solution

Use a shared user-defined network:

```bash
docker network create app-net

docker run -d --name db --network app-net postgres
docker run -d --name api --network app-net my-api
```

Configure:

```text
DB_HOST=db
DB_PORT=5432
```

---

## Q119. Scenario: PostgreSQL data disappears after recreating the container

### Root cause

The data was stored in the container writable layer.

### Solution

Use a named volume:

```bash
docker volume create pgdata

docker run -d \
  --name postgres \
  -v pgdata:/var/lib/postgresql/data \
  postgres
```

---

## Q120. Scenario: Docker image is 2 GB

### Investigation

```bash
docker history myapp:latest
docker image inspect myapp:latest
```

Look for:

- Large base image
- Build tools in final image
- Unnecessary packages
- Large build context
- Unnecessary application files

Use multistage builds, a suitable base image, and `.dockerignore`.

---

## Q121. Scenario: Docker build is slow

Investigate:

```text
Build context size
       ↓
.dockerignore
       ↓
Dockerfile instruction ordering
       ↓
Cache invalidation
       ↓
Dependency installation
       ↓
External package/registry downloads
```

A common optimization is:

```dockerfile
COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build
```

---

## Q122. Scenario: Container exits with 137

### Answer approach

```bash
docker inspect <container>
docker stats
dmesg | grep -i oom
journalctl -k | grep -i oom
```

Consider:

- Container memory limit
- Host memory pressure
- Explicit SIGKILL

Do not automatically conclude "137 = OOM."

---

## Q123. Scenario: DNS works but application connection fails

If:

```bash
getent hosts db
```

works, DNS resolution is functioning.

Next test:

```bash
nc -vz db 5432
```

If TCP fails, investigate network path, listening port, firewall, or service state rather than only DNS.

---

## Q124. Scenario: CI builds a new image but deployment runs old code

Possible causes:

```text
Wrong image tag
      ↓
Push failed
      ↓
Wrong registry/repository
      ↓
Mutable tag reused
      ↓
Deployment reused cached image
```

Use traceable version tags such as:

```text
myapp:<git-commit-sha>
```

and/or an immutable digest.

---

## Q125. Scenario: Docker socket is mounted into a CI container

### Question

Why is this risky?

### Answer

The container can communicate with the host Docker daemon. If the CI container is compromised, the attacker may obtain extensive control over Docker-managed resources and potentially the host.

Use a security design appropriate to the CI architecture rather than assuming the socket is harmless.

---

# 19. Docker vs Kubernetes

## Q126. Is Docker the same as Kubernetes?

No.

Docker is a container platform/ecosystem. Kubernetes is a container orchestration platform.

They address different layers of the problem.

---

## Q127. Why use Kubernetes if Docker can run containers?

Docker Engine can run containers on individual hosts.

Kubernetes adds cluster-level capabilities such as:

- Scheduling
- Desired-state management
- Self-healing
- Service discovery
- Rolling updates
- Scaling
- Declarative workload management

---

## Q128. Does Kubernetes require Docker Engine?

No.

Modern Kubernetes commonly uses CRI-compatible runtimes such as:

- containerd
- CRI-O

Kubernetes no longer requires Docker Engine as its container runtime.

---

## Q129. Docker vs Kubernetes

| Docker Engine | Kubernetes |
|---|---|
| Container platform/runtime ecosystem | Container orchestrator |
| Commonly used directly on a host | Designed around cluster management |
| Docker CLI/API | Kubernetes API |
| Manages containers/images/networks/volumes | Manages workloads, scheduling, services, rollouts, scaling |
| Compose is common for local multi-container apps | Kubernetes resources are used for cluster workloads |

---

# 20. Rapid-Fire Questions

| Question | Answer |
|---|---|
| Docker? | Platform for building, distributing, and running containers |
| Container? | Isolated process created from an image |
| Image? | Immutable layered package |
| VM vs container? | Guest OS vs isolated process sharing host kernel |
| `dockerd`? | Docker Engine daemon |
| Docker CLI? | User-facing Docker client |
| `containerd`? | Container runtime component |
| `runc`? | Low-level OCI runtime |
| Namespace? | Resource isolation |
| cgroup? | Resource control/accounting |
| OCI? | Open container standards |
| PID 1? | First process in container PID namespace |
| Layer? | Filesystem change in image |
| Tag? | Mutable human-readable image reference |
| Digest? | Content-addressed image reference |
| `FROM`? | Base image/build stage |
| `RUN`? | Build-time command |
| `CMD`? | Default runtime command/args |
| `ENTRYPOINT`? | Primary runtime executable |
| `COPY`? | Copy files into image |
| `ADD`? | Copy plus additional behavior |
| `ARG`? | Build-time variable |
| `ENV`? | Container environment variable |
| `WORKDIR`? | Working directory |
| `USER`? | Default user/group |
| `EXPOSE`? | Documents intended container port |
| `-p`? | Publishes host/container port |
| Volume? | Docker-managed persistent storage |
| Bind mount? | Host path mounted into container |
| Registry? | Image storage/distribution |
| Compose? | Declarative multi-container tool |
| Swarm? | Docker-native orchestration |
| Exit 137? | SIGKILL; often OOM-related |
| `docker logs`? | Container logs |
| `docker inspect`? | Detailed object configuration/state |
| `docker exec`? | New process inside running container |

---

# 21. Common Interview Traps

## Trap 1: `EXPOSE` publishes a port

**Wrong.**

```dockerfile
EXPOSE 8080
```

documents the intended container port.

Publishing requires:

```bash
docker run -p 8080:8080 myapp
```

---

## Trap 2: `RUN` starts the application

**Wrong.**

```text
RUN → image build
CMD/ENTRYPOINT → container startup
```

---

## Trap 3: `CMD` and `ENTRYPOINT` are identical

**Wrong.**

`CMD` provides defaults; `ENTRYPOINT` defines the primary executable.

---

## Trap 4: `COPY` and `ADD` are identical

**Wrong.**

`ADD` has additional behavior. Prefer `COPY` for ordinary copying.

---

## Trap 5: `ARG` is automatically a runtime environment variable

**Wrong.**

`ARG` is primarily build-time. `ENV` is for the container environment.

---

## Trap 6: Containers have their own kernel

Normally wrong for Linux containers.

They normally share the host Linux kernel.

---

## Trap 7: `localhost` means the Docker host

**Wrong.**

Inside a container:

```text
localhost → current network namespace/container
```

---

## Trap 8: Container = lightweight VM

**Wrong.**

A normal container is an isolated process environment, not a complete guest OS.

---

## Trap 9: Removing a container removes its named volume

**Normally wrong.**

The named volume remains until it is explicitly removed.

---

## Trap 10: Exit 137 always means OOM

**Too absolute.**

137 corresponds to a process terminated with SIGKILL; OOM is a common cause.

---

## Trap 11: Restarting a container fixes the problem

Restarting may hide the symptom temporarily. Find the root cause.

---

## Trap 12: `depends_on` means the dependency is ready

**Wrong.**

Startup ordering is not the same as application readiness.

---

## Trap 13: Docker socket access is harmless

**Wrong.**

Docker daemon access is highly privileged.

---

## Trap 14: `docker commit` is the normal production build process

**Wrong.**

Use reproducible Dockerfiles and CI/CD builds.

---

# 22. Important Commands Cheat Sheet

## Information

```bash
docker version
docker info
docker context ls
docker system df
```

## Images

```bash
docker images
docker pull nginx:1.27
docker build -t myapp:1.0 .
docker tag myapp:1.0 registry/myapp:1.0
docker push registry/myapp:1.0
docker rmi myapp:1.0
docker image inspect myapp:1.0
docker history myapp:1.0
```

## Containers

```bash
docker run -d --name web nginx
docker create --name web nginx
docker start web
docker stop web
docker restart web
docker kill web
docker rm web
docker rm -f web
docker ps
docker ps -a
docker logs web
docker logs -f web
docker exec -it web sh
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
docker container prune
docker image prune
docker network prune
docker volume prune
docker system prune
docker system df -v
```

---

# 23. Final Interview Checklist

## Fundamentals

- [ ] Docker definition
- [ ] Container definition
- [ ] Image vs container
- [ ] Container vs VM
- [ ] Why containers are lightweight
- [ ] CI/CD use of images

## Architecture

- [ ] Docker CLI
- [ ] Docker API
- [ ] `dockerd`
- [ ] `containerd`
- [ ] OCI runtime / `runc`
- [ ] Docker socket
- [ ] `docker run` flow

## Internals

- [ ] Namespaces
- [ ] cgroups
- [ ] Namespace vs cgroup
- [ ] OCI
- [ ] PID 1

## Images

- [ ] Image
- [ ] Layers
- [ ] Build cache
- [ ] Copy-on-write
- [ ] Tags
- [ ] Digests

## Dockerfile

- [ ] `FROM`
- [ ] `RUN`
- [ ] `CMD`
- [ ] `ENTRYPOINT`
- [ ] `COPY`
- [ ] `ADD`
- [ ] `ARG`
- [ ] `ENV`
- [ ] `WORKDIR`
- [ ] `USER`
- [ ] `EXPOSE`
- [ ] `VOLUME`
- [ ] `LABEL`
- [ ] `HEALTHCHECK`
- [ ] `SHELL`
- [ ] `STOPSIGNAL`
- [ ] `ONBUILD`
- [ ] `.dockerignore`
- [ ] Build context
- [ ] Shell vs exec form

## Builds

- [ ] Multistage builds
- [ ] `COPY --from`
- [ ] Image optimization
- [ ] Cache optimization

## Commands

- [ ] `docker run`
- [ ] `docker create`
- [ ] `docker start`
- [ ] `docker stop`
- [ ] `docker kill`
- [ ] `docker exec`
- [ ] `docker logs`
- [ ] `docker inspect`
- [ ] `docker stats`
- [ ] `docker diff`
- [ ] `docker history`

## Networking

- [ ] Bridge
- [ ] Host
- [ ] None
- [ ] Overlay
- [ ] User-defined networks
- [ ] Docker DNS
- [ ] Port publishing
- [ ] `EXPOSE` vs `-p`
- [ ] `localhost`
- [ ] Network troubleshooting

## Storage

- [ ] Writable layer
- [ ] Volume
- [ ] Bind mount
- [ ] Persistence
- [ ] Volume lifecycle

## Configuration

- [ ] `ARG`
- [ ] `ENV`
- [ ] Runtime overrides
- [ ] Secret handling

## Registry

- [ ] Registry
- [ ] ECR
- [ ] Pull
- [ ] Push
- [ ] Tagging
- [ ] Digests
- [ ] CI/CD traceability

## Resources

- [ ] CPU limits
- [ ] Memory limits
- [ ] cgroups
- [ ] OOM investigation
- [ ] Disk cleanup

## Lifecycle

- [ ] Created
- [ ] Running
- [ ] Exited
- [ ] Restart policies
- [ ] Main process behavior

## Troubleshooting

- [ ] Container exits
- [ ] Restart loop
- [ ] Port failure
- [ ] DNS failure
- [ ] Container-to-container failure
- [ ] Internet connectivity
- [ ] Permission issues
- [ ] Daemon issues
- [ ] Disk exhaustion
- [ ] Memory pressure

## Orchestration

- [ ] Compose
- [ ] `depends_on`
- [ ] Readiness vs startup ordering
- [ ] Swarm
- [ ] Scaling

## Advanced

- [ ] `docker commit`
- [ ] `docker export`
- [ ] `docker save`
- [ ] Docker socket security
- [ ] Runtime vs orchestrator

## Scenarios

- [ ] Container exits immediately
- [ ] Container restarts continuously
- [ ] Port is inaccessible
- [ ] API cannot reach DB
- [ ] Persistent data disappears
- [ ] Image is too large
- [ ] Build is too slow
- [ ] Exit code 137
- [ ] DNS works but TCP fails
- [ ] Old image deployed
- [ ] Docker socket security

---

# Interview Answer Formula

For a conceptual question:

```text
1. Define it
2. Explain why it exists
3. Explain how it works
4. Give an example
5. State an important limitation/trap
```

For a troubleshooting question:

```text
1. Identify the symptom
2. Check state
3. Check logs
4. Check configuration
5. Check networking/storage/resources
6. Identify root cause
7. Fix it
8. Verify the fix
9. Explain prevention
```

## Final Mental Model

```text
Docker CLI
    ↓
Docker API
    ↓
dockerd
    ↓
containerd
    ↓
OCI runtime
    ↓
Linux kernel
    ├── namespaces → isolation
    └── cgroups    → resource control
             ↓
         Container
             ↓
       Main Process
```

If you understand this flow, Dockerfile behavior, image layers, networking, storage, lifecycle, resource management, and troubleshooting become much easier to reason about.
