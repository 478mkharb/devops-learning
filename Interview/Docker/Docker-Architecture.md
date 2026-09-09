# Docker Interview Preparation — Topic 2: Docker Architecture

> **Goal:** Understand what happens internally when you execute a Docker command, and how the Docker CLI, daemon, containerd, OCI runtime, and Linux kernel fit together.
>
> **Interview focus:** Do not memorize component names only. Be able to explain the request flow and the responsibility of each layer.

---

## Q13. What is Docker architecture?

### Short Interview Answer

Docker uses a client-server architecture in which the **Docker CLI communicates with the Docker Engine API**, and the Docker daemon (`dockerd`) manages images, containers, networks, and volumes. Modern Docker Engine also uses **containerd** and an **OCI runtime** such as `runc` to create and manage container processes.

### High-Level Architecture

```text
Docker CLI
    │
    │ Docker Engine API
    ▼
 dockerd
    │
    ▼
containerd
    │
    ▼
OCI Runtime
(commonly runc)
    │
    ▼
Linux Kernel
(namespaces + cgroups)
    │
    ▼
Container Process
```

### What Does Each Component Do?

| Component | Main Responsibility |
|---|---|
| Docker CLI | User-facing command-line client |
| Docker Engine API | API interface used to request Docker operations |
| `dockerd` | Docker daemon / engine management |
| containerd | Container lifecycle and runtime management |
| OCI runtime | Creates/starts the actual container process |
| Linux kernel | Provides isolation and resource-control primitives |

### Important Nuance

This is a **conceptual architecture**. Exact implementation details can vary by Docker release and platform.

For interview purposes, the important idea is:

```text
CLI
 ↓
Docker Engine
 ↓
containerd
 ↓
OCI runtime
 ↓
Linux kernel
```

### Interview Point

If asked:

> "Is Docker CLI the Docker engine?"

Answer:

**No.** The CLI is a client that sends requests to the Docker Engine API. The daemon performs the actual management work.

---

## Q14. What is Docker Engine?

### Short Interview Answer

Docker Engine is the core Docker platform that provides the API and daemon used to build and manage Docker images, containers, networks, and volumes.

### Main Pieces

Conceptually:

```text
Docker Engine
 ├── Docker API
 └── Docker daemon (`dockerd`)
```

The Docker CLI is normally a separate client that communicates with the Engine API.

### Example

When you execute:

```bash
docker ps
```

the CLI does not itself inspect the kernel and discover containers.

Instead, the request follows the Docker Engine interface:

```text
docker ps
   │
   ▼
Docker API
   │
   ▼
dockerd
   │
   ▼
Container/runtime state
   │
   ▼
Result returned to CLI
```

### Why This Matters

Understanding the API boundary explains why the Docker CLI can communicate with a Docker daemon running somewhere else.

For example:

```text
Docker CLI
    │
    │ API
    ▼
Remote Docker Engine
    │
    └── Containers
```

### Interview Trap

**Wrong:** "Docker Engine is just the Docker CLI."

The CLI is the client. The Engine provides the daemon/API functionality that performs Docker operations.

---

## Q15. What is Docker CLI?

### Short Interview Answer

The Docker CLI is the command-line client used to send commands to the Docker Engine.

### Example

When you run:

```bash
docker run nginx
```

the CLI interprets your command and sends the corresponding request to the Docker Engine.

Conceptually:

```text
User
 │
 │ docker run nginx
 ▼
Docker CLI
 │
 │ API request
 ▼
Docker Engine
```

### Common Commands

```bash
docker ps
docker images
docker pull nginx
docker run nginx
docker stop nginx
docker rm nginx
```

These commands are interfaces to Docker's management functionality.

### Important Point

The CLI is not responsible for directly creating Linux namespaces or cgroups.

Those lower-level operations happen through the Docker Engine/container runtime stack.

### Interview Point

Think:

> **CLI = client/interface**

not:

> **CLI = container runtime**

---

## Q16. What is `dockerd`?

### Short Interview Answer

`dockerd` is the **Docker daemon** that listens for Docker API requests and manages Docker objects such as containers, images, networks, and volumes.

### Architecture Position

```text
docker CLI
    │
    ▼
Docker API
    │
    ▼
 dockerd
    │
    ▼
containerd
```

### Example

When you execute:

```bash
docker run nginx
```

`dockerd` coordinates the requested operation.

Depending on what is required, this may involve:

- Obtaining the image
- Creating container configuration
- Setting up networking
- Setting up storage
- Asking containerd/runtime to create the container process

### Why Is `dockerd` Important?

If the Docker daemon is not available, ordinary Docker CLI operations cannot be performed against that engine.

For example:

```bash
docker info
```

can be useful when diagnosing whether the Docker Engine is reachable.

On a Linux host using systemd, you can also inspect:

```bash
systemctl status docker
```

### Interview Trap

**Wrong:** "`dockerd` is the container."

No.

```text
dockerd → daemon managing Docker
container → isolated application runtime
```

### Interview Point

Remember:

> **`dockerd` is the Docker daemon, not the container runtime itself.**

---

## Q17. What is containerd?

### Short Interview Answer

containerd is a **container runtime/lifecycle management component** used by Docker Engine to manage container execution and related lifecycle operations.

### Position in the Stack

```text
Docker CLI
     ↓
dockerd
     ↓
containerd
     ↓
OCI runtime
     ↓
Linux kernel
```

### Why Was containerd Introduced?

Docker's architecture separates higher-level Docker management from lower-level container lifecycle responsibilities.

containerd can manage things such as:

- Container lifecycle
- Image-related operations
- Container execution
- Runtime integration

The exact responsibilities exposed through a particular Docker version can evolve, but the architectural separation is important.

### Is containerd the Same as Docker?

No.

Docker Engine is a broader platform.

containerd is a lower-level component used by Docker and also used directly by other container platforms.

### Docker vs containerd

```text
Docker
 ├── Docker CLI/API
 ├── dockerd
 ├── networking
 ├── volumes
 └── container management

containerd
 └── container lifecycle/runtime management
```

### Interview Trap

**Wrong:** "containerd is a Kubernetes-only component."

containerd is a general-purpose container runtime component and can be used outside Kubernetes.

### Interview Point

A good answer:

> "Docker Engine uses containerd as a lower-level component for container lifecycle management."

---

## Q18. What is an OCI runtime?

### Short Interview Answer

An OCI runtime is software that implements the **Open Container Initiative runtime specification** and is responsible for creating and starting container processes according to the runtime configuration.

A commonly used OCI runtime is **`runc`**.

### Architecture

```text
dockerd
   ↓
containerd
   ↓
OCI runtime
   ↓
Linux kernel
```

### What Does the Runtime Actually Do?

At a conceptual level, the OCI runtime takes the container configuration and creates the isolated process environment.

That involves Linux kernel features such as:

- Namespaces
- cgroups
- Mounts
- Capabilities
- Process isolation

### Example

A simplified mental model is:

```text
Container configuration
        ↓
OCI runtime
        ↓
Create isolated process
        ↓
Linux kernel mechanisms
```

### `runc`

`runc` is a widely used OCI-compliant runtime.

Do not confuse:

```text
Docker
containerd
runc
```

They are different layers.

### Interview Trap

**Wrong:** "runc is the Docker daemon."

No.

```text
dockerd → Docker daemon
containerd → lifecycle/runtime management
runc → OCI runtime
```

### Interview Point

The runtime is closer to the kernel-level container creation operation than the Docker CLI or Docker daemon.

---

## Q19. What is the difference between Docker, containerd, and runc?

### Short Interview Answer

They operate at different layers:

- **Docker Engine** provides the higher-level container platform and management interface.
- **containerd** handles lower-level container lifecycle/runtime management.
- **runc** is an OCI runtime that creates and starts the container process.

### Comparison

| Component | Layer | Primary Role |
|---|---|---|
| Docker | Higher-level platform | Build/manage/run containers, images, networks, volumes |
| containerd | Container lifecycle layer | Manage container lifecycle and runtime integration |
| runc | OCI runtime | Create/start container process using Linux kernel features |

### Mental Model

```text
             Docker
        ┌───────────────┐
        │ CLI / API     │
        │ dockerd       │
        └───────┬───────┘
                │
                ▼
            containerd
                │
                ▼
              runc
                │
                ▼
          Linux Kernel
```

### Why Is This Separation Useful?

Each layer has a more focused responsibility.

For example:

```text
User request
    ↓
Docker understands the request
    ↓
containerd manages lifecycle
    ↓
runc creates the isolated process
    ↓
kernel provides isolation/resources
```

### Interview Trap

Don't describe all three as interchangeable "Docker runtimes."

They are related, but they have different responsibilities.

### Interview Point

If you can draw this stack from memory, you can answer many Docker architecture follow-ups:

```text
CLI → dockerd → containerd → runc → kernel
```

---

## Q20. What happens internally when you run `docker run nginx`?

### Short Interview Answer

Docker receives the request through the CLI/API, checks or obtains the image, creates the container configuration and required resources, and uses containerd and an OCI runtime to create/start the container process.

### Step-by-Step Flow

Suppose you run:

```bash
docker run nginx
```

Conceptually:

```text
1. docker CLI
       ↓
2. Docker Engine API
       ↓
3. dockerd
       ↓
4. Check image
       ↓
5. Pull image if required
       ↓
6. Prepare container filesystem/config
       ↓
7. Set up networking
       ↓
8. containerd
       ↓
9. OCI runtime
       ↓
10. Linux kernel
       ↓
11. nginx process starts
```

### Step 1 — CLI

The CLI receives:

```bash
docker run nginx
```

and sends the request to the Docker Engine.

### Step 2 — Image Resolution

Docker needs the `nginx` image.

If it is not locally available, Docker obtains it from a registry according to the image reference.

Conceptually:

```text
Local image?
   │
 ┌─┴─┐
Yes  No
 │    │
 │    └── Pull image
 ▼
Continue
```

### Step 3 — Container Creation

Docker creates the container's configuration and filesystem view based on the image.

### Step 4 — Networking

Docker sets up the container's network according to the requested/default networking configuration.

### Step 5 — Runtime

The request moves down the stack:

```text
dockerd
  ↓
containerd
  ↓
OCI runtime
```

### Step 6 — Kernel Isolation

The runtime asks the Linux kernel to create the process environment using mechanisms such as namespaces and cgroups.

### Step 7 — Application Starts

The image's configured/default startup process is launched.

For the official NGINX image, the NGINX process becomes the main process for the container.

### Important Interview Point

The command:

```bash
docker run nginx
```

does **not** mean:

> "Docker copies an entire VM and boots it."

Instead, Docker creates an isolated process environment from the image and starts the application's process.

---

## Q21. What are Linux namespaces and why are they important for Docker?

### Short Interview Answer

Linux namespaces provide **isolation of system resources and views** so that processes inside a container can have a different view of processes, networking, mounts, hostnames, users, and other kernel resources.

### Why Docker Needs Them

Without isolation, a process in one container could see much more of the host environment.

Namespaces allow Docker to create separate views.

For example:

```text
Host PID namespace
 ├── PID 1
 ├── PID 200
 ├── PID 500
 └── ...

Container PID namespace
 ├── PID 1
 ├── PID 2
 └── ...
```

The container's PID 1 is not necessarily PID 1 on the host.

### Common Namespace Types

| Namespace | Provides isolation for |
|---|---|
| PID | Process IDs/process visibility |
| NET | Network interfaces, routes, ports |
| MNT | Mount/filesystem view |
| UTS | Hostname/domain name |
| IPC | Inter-process communication resources |
| USER | User/group ID mappings |

Linux versions and container configurations can also involve other namespaces, such as the cgroup namespace.

### Example: PID Namespace

Inside a container:

```bash
ps
```

may show only processes belonging to that container's PID namespace.

On the host:

```bash
ps aux
```

shows the host's broader process view.

### Example: Network Namespace

A container can have its own:

```text
eth0
IP address
routing table
network namespace
```

while the host has its own network namespace.

### Important Point

Namespaces provide **isolation**, not resource limits.

That distinction matters.

```text
Namespaces → "What can I see/use?"
cgroups    → "How much can I use?"
```

### Interview Point

If asked why containers are isolated:

> **"Linux namespaces isolate the container's view of kernel resources and cgroups control/account resource usage."**

---

## Q22. What are cgroups and why are they important for Docker?

### Short Interview Answer

Linux **control groups (cgroups)** provide mechanisms for controlling and accounting for resource usage by groups of processes, including CPU, memory, and process-count limits.

### Namespaces vs cgroups

This is one of the most important Docker interview distinctions.

```text
Namespaces
    ↓
Isolation / visibility

cgroups
    ↓
Resource control / accounting
```

### Example

Suppose a container should use at most:

```text
Memory → 512 MB
CPU    → Limited share/quota
PIDs   → Limited number of processes
```

Docker can configure corresponding resource controls.

Conceptually:

```text
Container
    │
    ├── Namespace isolation
    │
    └── cgroup
         ├── CPU
         ├── Memory
         └── PIDs
```

### Why This Matters

Without resource controls, one workload could potentially consume excessive host resources and affect other workloads.

Resource controls help enforce workload boundaries.

### Example

```bash
docker run --memory=512m nginx
```

This requests a memory limit for the container.

You can inspect container resource usage with:

```bash
docker stats
```

### Important Nuance

A cgroup is not a VM boundary.

It controls resource usage of processes; it does not provide the same isolation model as a hypervisor.

### Interview Trap

**Wrong:**

> "Namespaces limit CPU and memory."

Namespaces are primarily about isolation/visibility. cgroups provide resource control/accounting.

### Interview Point

Memorize:

```text
Namespace = isolation
cgroup    = resource control
```

---

## Q23. What is the role of the Linux kernel in Docker containers?

### Short Interview Answer

The Linux kernel provides the low-level primitives that make Linux containers possible, including **namespaces, cgroups, capabilities, filesystem isolation, networking, and process management**.

### Important Concept

Docker itself does not implement an entirely new operating-system kernel for every container.

Instead:

```text
Container A ─┐
Container B ─┼──→ Linux Kernel
Container C ─┘
```

The kernel provides the underlying mechanisms.

### Examples

Docker/container runtime relies on kernel features for:

**Process isolation**

```text
PID namespaces
```

**Network isolation**

```text
Network namespaces
```

**Resource control**

```text
cgroups
```

**Filesystem isolation**

```text
Mount namespaces
```

**Privilege reduction**

```text
Linux capabilities
```

### Why This Explains Container Efficiency

Because multiple containers can use the same kernel:

```text
One kernel
   │
 ┌─┼───────────────┐
 ▼ ▼               ▼
C1 C2              C3
```

there is no need for a separate guest kernel per container.

### Important Limitation

Linux containers depend on Linux kernel functionality.

This is also why running Linux containers on macOS or Windows generally requires a Linux environment, often provided through virtualization by Docker Desktop.

### Interview Point

The container runtime is the layer that asks the kernel to create the required isolation and process environment.

---

## Q24. What is the difference between Docker Engine and Docker Desktop?

### Short Interview Answer

**Docker Engine** is the core container engine/daemon and API. **Docker Desktop** is a developer-focused application that packages Docker functionality with additional tooling and, on macOS/Windows, provides the Linux environment needed for Linux containers.

### Linux

On a Linux host, Docker Engine can run directly on the Linux kernel:

```text
Linux Host
   ↓
Docker Engine
   ↓
Containers
```

### macOS/Windows

For Linux containers:

```text
macOS / Windows
       ↓
Docker Desktop
       ↓
Linux environment / VM
       ↓
Docker Engine
       ↓
Linux Containers
```

### What Docker Desktop Adds

Depending on platform/version, Docker Desktop can provide:

- Docker Engine
- Docker CLI integration
- Container/image management UI
- Docker Compose
- Development tooling
- Linux VM/environment on platforms that need it

### Interview Trap

**Wrong:** "Docker Desktop is the Docker container runtime."

Docker Desktop is a desktop application/platform bundle. The actual container execution involves Docker Engine and the underlying runtime stack.

### Interview Point

Think:

```text
Docker Engine → core engine
Docker Desktop → developer desktop product/bundle
```

---

## Q25. What is the difference between Docker daemon and Docker client?

### Short Interview Answer

The **Docker client** (CLI) sends commands/API requests. The **Docker daemon (`dockerd`)** receives those requests and performs the management operations.

### Architecture

```text
User
 │
 │ docker run nginx
 ▼
Docker Client / CLI
 │
 │ Docker Engine API
 ▼
Docker Daemon
 (`dockerd`)
 │
 ├── Images
 ├── Containers
 ├── Networks
 └── Volumes
```

### Why Is This Important?

The client and daemon do not have to be the same process or even run on the same machine.

Conceptually:

```text
Laptop
Docker CLI
    │
    │ API
    ▼
Remote Server
Docker Engine
    │
    └── Containers
```

### Example Use Case

A developer can use a Docker client to communicate with a remote Docker Engine.

This makes the client-server architecture useful for:

- Remote administration
- Automation
- CI/CD
- Tool integrations

### Interview Trap

**Wrong:** "The Docker CLI creates containers directly."

The CLI sends the request; the Docker Engine/daemon coordinates the operation.

### Interview Point

Simple rule:

> **Client asks; daemon performs.**

---

## Q26. What is the Docker Engine API?

### Short Interview Answer

The Docker Engine API is the API through which clients communicate with the Docker daemon to perform operations such as creating containers, managing images, and inspecting resources.

### Architecture

```text
Docker CLI
    │
    │ HTTP/API
    ▼
Docker Engine API
    │
    ▼
dockerd
```

The CLI is therefore one client of the Docker Engine API.

Other tools can also interact with Docker through APIs or supported integrations.

### Why APIs Matter

Automation systems can use the Docker Engine interface rather than manually typing commands.

For example:

```text
CI/CD Tool
     │
     ▼
Docker Engine API
     │
     ▼
Docker Engine
```

### Security Consideration

Access to a Docker Engine API/socket can be highly privileged.

On Linux, Docker commonly exposes a Unix socket such as:

```text
/var/run/docker.sock
```

Giving a process access to the Docker socket should therefore be treated as granting significant control over the host's Docker environment.

### Interview Trap

Do not describe the Docker API as:

> "The API inside the container."

It is the management interface for the Docker Engine.

### Interview Point

```text
CLI → API → dockerd
```

is the key relationship.

---

## Q27. What happens if the Docker daemon is stopped?

### Short Interview Answer

If the Docker daemon is unavailable, Docker CLI operations that require communication with that engine generally fail. Existing containers may continue running because their processes are managed through lower-level runtime components, but the exact behavior depends on the Docker/runtime architecture and operation.

### Example

If the daemon is unavailable and you run:

```bash
docker ps
```

you may receive an error indicating that the Docker daemon cannot be reached.

### Important Distinction

Do not automatically assume:

> "Docker daemon stops → every container immediately stops."

That is too simplistic.

Docker Engine has a higher-level management role, while container processes are ultimately handled through lower-level runtime components.

### Practical Troubleshooting

Check:

```bash
systemctl status docker
```

Then:

```bash
docker info
```

You can also inspect daemon logs:

```bash
journalctl -u docker
```

### Interview Point

Separate:

```text
Docker management plane
        ↓
dockerd

Container execution stack
        ↓
containerd / OCI runtime
```

This distinction explains why daemon availability and container process execution are related but not identical concepts.

---

# Topic 2 — Architecture Revision

## The Core Stack

Memorize this diagram:

```text
┌───────────────────────────┐
│        Docker CLI         │
│       docker command      │
└─────────────┬─────────────┘
              │
              │ Docker Engine API
              ▼
┌───────────────────────────┐
│          dockerd          │
│      Docker daemon        │
└─────────────┬─────────────┘
              │
              ▼
┌───────────────────────────┐
│        containerd         │
│ Container lifecycle layer │
└─────────────┬─────────────┘
              │
              ▼
┌───────────────────────────┐
│       OCI runtime         │
│       e.g. runc           │
└─────────────┬─────────────┘
              │
              ▼
┌───────────────────────────┐
│       Linux Kernel        │
│ namespaces + cgroups etc. │
└───────────────────────────┘
```

## Namespace vs cgroup

This distinction is extremely important:

| Mechanism | Main Purpose | Example |
|---|---|---|
| Namespaces | Isolation / visibility | Container sees its own PID/network namespace |
| cgroups | Resource control/accounting | Limit memory or CPU |
| Capabilities | Fine-grained privilege control | Remove unnecessary Linux privileges |

---

# What Happens During `docker run`?

Use this answer in an interview:

```text
docker run nginx
       │
       ▼
Docker CLI
       │
       ▼
Docker Engine API
       │
       ▼
dockerd
       │
       ├── Image management
       ├── Network setup
       └── Container configuration
       │
       ▼
containerd
       │
       ▼
OCI runtime (e.g. runc)
       │
       ▼
Linux kernel
       │
       ├── namespaces
       ├── cgroups
       ├── mounts
       └── capabilities
       │
       ▼
NGINX process
```

---

# High-Value Architecture Interview Traps

### Trap 1 — Docker CLI = Docker Engine

❌ No.

The CLI is a client.

```text
CLI → Docker API → Engine
```

### Trap 2 — dockerd = container runtime

❌ Not exactly.

`dockerd` is the Docker daemon. Modern Docker Engine uses lower-level components such as containerd and an OCI runtime.

### Trap 3 — containerd = Kubernetes runtime only

❌ No.

containerd is a general-purpose container runtime/lifecycle component.

### Trap 4 — runc = Docker

❌ No.

`runc` is an OCI runtime used to create/start container processes.

### Trap 5 — namespaces control CPU and memory

❌ No.

```text
Namespaces → isolation
cgroups    → resource control
```

### Trap 6 — Docker creates a mini VM

❌ No.

For Linux containers:

```text
Container
   ↓
Host Linux Kernel
```

rather than:

```text
Container
   ↓
Guest OS + Guest Kernel
```

### Trap 7 — Docker daemon stops, containers must immediately die

⚠️ Too simplistic.

Docker daemon availability and container process execution involve different layers of the runtime stack.

---

# Interview Follow-Up Questions

After explaining Docker architecture, an interviewer may immediately ask:

1. What is containerd?
2. What is `runc`?
3. What is OCI?
4. What are Linux namespaces?
5. What are cgroups?
6. What is the difference between namespaces and cgroups?
7. What happens internally during `docker run`?
8. What happens if `dockerd` goes down?
9. Is Docker required by Kubernetes?
10. Does Kubernetes use Docker Engine as its container runtime?
11. Why can Docker run containers without a VM on Linux?
12. Why does Docker Desktop use a VM on macOS/Windows?
13. What is the Docker Engine API?
14. What is `/var/run/docker.sock`?
15. Why is access to the Docker socket considered highly privileged?

---

# Topic 2 — Final Mental Model

If you remember only one thing from this topic, remember:

```text
             USER
              │
              ▼
        Docker CLI
              │
              ▼
     Docker Engine API
              │
              ▼
           dockerd
              │
              ▼
         containerd
              │
              ▼
       OCI runtime
        (e.g. runc)
              │
              ▼
        Linux Kernel
       /     |      \
 namespaces cgroups capabilities
       \     |      /
              ▼
       Container Process
```

### The simplest interview explanation

> **"When I run a Docker command, the Docker CLI sends the request to the Docker Engine API. The Docker daemon coordinates the operation, containerd handles lower-level container lifecycle management, and an OCI runtime such as runc creates the container process using Linux kernel mechanisms such as namespaces and cgroups."**

That single explanation connects almost every major concept in Docker architecture.
