# Docker Interview Preparation — Topic 1: Docker Fundamentals

> **Goal:** Build an interview-ready understanding of Docker fundamentals, not just memorized definitions.
>
> For each question, focus on **what it means, how it works, why it matters, and the common interview trap**.

---

## Q1. What is Docker?

### Short Interview Answer

Docker is a platform for **building, packaging, distributing, and running applications in containers**.

A Docker image packages an application together with its required dependencies and configuration. Docker then uses that image to create isolated containers in which the application runs.

### Detailed Explanation

Without containerization, an application may depend on:

- A particular operating system environment
- Specific library versions
- Runtime versions
- Environment variables
- System packages
- Application dependencies

This can create the classic problem:

> "It works on my machine, but not on the server."

Docker addresses this by packaging the application and its user-space dependencies into an **image**.

That image can then be used consistently across environments:

```text
Developer Machine
       │
       │ Docker Image
       ▼
   Test Environment
       │
       │ Same Image
       ▼
   Production
```

The important point is that Docker does **not** package a complete guest operating system in every container. Containers normally share the host's kernel while remaining isolated from one another.

### Simple Example

Suppose we have a Python application:

```text
Python Application
       +
Python runtime
       +
Python dependencies
       +
Application files
       +
Configuration
       ↓
   Docker Image
       ↓
    Container
```

The same image can be used to start containers in development, testing, and production.

### Why Docker Matters in DevOps

Docker makes application delivery more predictable.

A CI/CD pipeline can:

1. Build the application.
2. Build a Docker image.
3. Test the image.
4. Push the image to a registry.
5. Deploy the same image to another environment.

This reduces differences between build and runtime environments.

### Common Interview Trap

**Wrong:** "Docker is a lightweight virtual machine."

Docker containers are **not VMs**. Containers provide process-level isolation and normally share the host kernel.

### Interview Point

Remember:

```text
Docker
  ↓
Build
  ↓
Package
  ↓
Distribute
  ↓
Run
  ↓
Application in containers
```

---

## Q2. What is containerization?

### Short Interview Answer

Containerization is the process of packaging an application and its required user-space dependencies into an isolated, portable runtime environment called a **container**.

### Detailed Explanation

Traditional deployment often installs an application directly onto a server:

```text
Server
 ├── OS
 ├── Runtime
 ├── Libraries
 ├── Application A
 └── Application B
```

Different applications may require different versions of libraries or runtimes, creating dependency conflicts.

With containerization:

```text
Host
 │
 ├── Container A
 │    └── Application A
 │
 ├── Container B
 │    └── Application B
 │
 └── Container C
      └── Application C
```

Each application gets its own user-space environment.

The containers are isolated using Linux kernel mechanisms such as **namespaces** and **cgroups**.

- **Namespaces** help isolate what a process can see.
- **cgroups** control and account for resources such as CPU and memory.

### Example

An organization might run:

```text
Container 1 → Python API
Container 2 → Java API
Container 3 → Redis
Container 4 → NGINX
```

All of them can run on the same host even though their applications use different runtimes.

### Why It Matters

Containerization improves:

- Portability
- Deployment consistency
- Application isolation
- CI/CD workflows
- Resource utilization
- Application packaging

### Common Interview Trap

Containerization does **not** mean that every container contains its own kernel.

Normally:

```text
Container A ─┐
Container B ─┼──→ Host Linux Kernel
Container C ─┘
```

### Interview Point

A good distinction is:

> **Containerization is the approach/process; Docker is one platform/tool used to implement it.**

---

## Q3. What is a Docker container?

### Short Interview Answer

A Docker container is a **runtime instance of a Docker image** with an isolated process environment and a writable container layer.

### Detailed Explanation

An image is a packaged application template.

When Docker starts a container from that image, Docker creates a runtime environment around the image.

Conceptually:

```text
Docker Image
 ├── Read-only Layer
 ├── Read-only Layer
 └── Read-only Layer
          │
          │ docker run
          ▼
      Container
          │
          └── Writable Layer
```

The image layers remain read-only. Changes made by the running container are normally written to the container's writable layer unless storage is mounted separately.

### Example

```bash
docker run nginx
```

This tells Docker to create and start a container using the `nginx` image.

You can then see it with:

```bash
docker ps
```

### Container ≠ Image

An image is the **package/template**.

A container is the **running or stopped runtime instance** created from that image.

You can create many containers from one image:

```text
             nginx image
             /    |    \
            /     |     \
           ▼      ▼      ▼
       nginx-1 nginx-2 nginx-3
```

Each container has its own runtime state.

### Important Point About Container Data

If you write data into the container's writable layer and then remove the container:

```bash
docker rm mycontainer
```

that writable-layer data is removed with the container.

For persistent data, use a:

- Docker volume
- Bind mount
- Appropriate external storage

### Common Interview Trap

**Wrong:** "A container is a small VM."

A container is better understood as an isolated process environment sharing the host kernel.

### Interview Point

Memorize this relationship:

```text
Image = package/template
Container = runtime instance of that image
```

---

## Q4. What is a Docker image?

### Short Interview Answer

A Docker image is an **immutable, layered package containing an application's filesystem content and metadata**, used as the template for creating containers.

### Detailed Explanation

A Docker image commonly consists of multiple filesystem layers.

For example:

```text
Application Layer
-----------------
Python dependencies
-----------------
Python runtime
-----------------
Ubuntu base filesystem
```

Each layer can be reused by other images when their contents match.

This layered model helps Docker avoid storing identical data repeatedly.

### Example

Consider:

```dockerfile
FROM python:3.12
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY app.py .
CMD ["python", "app.py"]
```

The resulting image contains the filesystem and metadata required to run the application.

### Image vs File Archive

An image is more than a simple application `.zip`.

It contains:

- Filesystem content
- Layer information
- Configuration metadata
- Default command information
- Environment metadata
- Other image configuration

### Why Images Are Called Immutable

Once an image version has been built, running containers do not normally modify the image layers themselves.

Instead:

```text
Image Layers
   ↓
Read-only

Container
   ↓
Writable runtime layer
```

If you change the Dockerfile and rebuild, Docker creates a new image result.

### Tags vs Digests

A tag such as:

```text
nginx:latest
```

is a human-friendly reference and can be moved to point to a different image.

A digest such as:

```text
nginx@sha256:...
```

is content-addressed and identifies a specific image manifest/content.

Therefore, for reproducible deployments, digests provide stronger immutability guarantees than mutable tags.

### Common Interview Trap

**Wrong:** "`latest` always means the newest image."

`latest` is just a tag. It does not inherently guarantee that the image is the newest version.

### Interview Point

Remember:

```text
Image
 ├── Layers
 ├── Filesystem content
 └── Metadata/configuration

        ↓

Container
```

---

## Q5. What is the difference between a Docker image and a container?

### Short Interview Answer

A **Docker image** is an immutable package/template, while a **container** is a runtime instance created from that image.

### Detailed Comparison

| Image | Container |
|---|---|
| Template/package | Runtime instance |
| Normally immutable | Has runtime state |
| Contains filesystem layers | Uses image + writable layer |
| Can be stored in a registry | Runs on a Docker host |
| Can create many containers | Represents one container instance |

### Example

Suppose we have:

```text
Image:
myapp:v1
```

We can create:

```text
myapp:v1
   │
   ├── Container A
   ├── Container B
   └── Container C
```

All three containers originate from the same image.

However, each container has its own:

- Process state
- Network identity
- Writable layer
- Runtime configuration

### Analogy

Think of:

```text
Image      → Class/template
Container  → Object/instance
```

This analogy is useful for interviews, although it is not a perfect technical equivalence.

### Common Interview Trap

Don't say:

> "The image is the running application."

The **container** is the runtime environment in which the application process executes.

### Interview Point

If an interviewer asks:

> "Can I have multiple containers from one image?"

Answer:

> **Yes. One image can be used to create many independent container instances.**

---

## Q6. What problem does Docker solve?

### Short Interview Answer

Docker reduces **environment inconsistency and dependency conflicts** by packaging an application and its user-space dependencies into a reproducible container image.

### The Traditional Problem

Consider three environments:

```text
Developer
Python 3.11
Library X v1

Testing
Python 3.12
Library X v2

Production
Python 3.10
Library X v1
```

The application may behave differently in each environment.

This creates:

- Dependency conflicts
- Configuration differences
- "Works on my machine" problems
- Difficult deployments
- Slow environment setup

### Docker's Approach

Package the application environment:

```text
Application
+
Runtime
+
Dependencies
+
Required user-space files
        ↓
   Docker Image
        ↓
Same artifact across environments
```

### CI/CD Example

A pipeline might perform:

```text
Git Commit
    ↓
Build
    ↓
Docker Image
    ↓
Automated Tests
    ↓
Registry
    ↓
Production
```

The important DevOps principle is:

> **Build once, promote the same artifact.**

Instead of rebuilding the application differently for each environment, the same tested image can be promoted.

### What Docker Does NOT Solve

Docker does not automatically solve:

- Application bugs
- Bad architecture
- Database consistency
- Network design
- Secrets management
- Monitoring
- Security configuration

Docker provides packaging and runtime isolation; the rest still requires proper engineering.

### Interview Point

A strong answer is:

> "Docker solves environment consistency and application packaging problems by creating a portable, reproducible runtime artifact."

---

## Q7. Why are Docker containers lightweight compared to virtual machines?

### Short Interview Answer

Containers are lightweight because they normally **share the host kernel** instead of running a complete guest operating system for every application.

### Virtual Machine Model

A VM typically looks like:

```text
Physical Host
     │
 Hypervisor
 ├───────────────┐
 │ VM 1          │
 │ Guest OS      │
 │ Application   │
 └───────────────┘
 ├───────────────┐
 │ VM 2          │
 │ Guest OS      │
 │ Application   │
 └───────────────┘
```

Each VM has its own guest OS kernel.

### Container Model

Containers normally look like:

```text
Physical/Virtual Host
        │
    Linux Kernel
    ├──────┬──────┬──────┐
    │      │      │      │
   C1     C2     C3     C4
   App    App    App    App
```

The containers share the kernel but have isolated process/filesystem/network views.

### Why This Reduces Overhead

A container does not normally need:

```text
Full guest OS
Guest kernel
Virtual hardware
```

for every application.

Instead, the host kernel provides the underlying kernel services.

### Important Nuance

"Lightweight" does **not** mean:

> "A container uses no memory."

Each application still consumes CPU, memory, file descriptors, network resources, etc.

The advantage is that the container does not require a separate full guest OS environment.

### Interview Point

The strongest one-line explanation:

> **VMs virtualize machines; containers isolate processes while sharing the host kernel.**

---

## Q8. What is the difference between a Docker container and a virtual machine?

### Short Interview Answer

A VM virtualizes a complete machine and normally includes a guest OS kernel. A container provides process-level isolation while normally sharing the host kernel.

### Comparison

| Feature | Container | Virtual Machine |
|---|---|---|
| Virtualizes | Application/process environment | Complete machine |
| Guest OS | Usually no separate guest kernel | Yes |
| Kernel | Shares host kernel | Guest kernel |
| Startup | Usually very fast | Usually slower |
| Resource overhead | Lower | Higher |
| Isolation boundary | Process/kernel mechanisms | Hypervisor/VM boundary |
| Typical use | Microservices, CI/CD | Strong OS isolation, different OS kernels |

### Architecture

**VM:**

```text
Hardware
   ↓
Hypervisor
   ↓
Guest OS
   ↓
Application
```

**Container:**

```text
Hardware
   ↓
Host OS Kernel
   ↓
Container Runtime
   ↓
Container
   ↓
Application
```

### Does a Container Have an OS?

This is a common interview trick.

A container has a **user-space filesystem/environment**, which may contain files from a Linux distribution such as Ubuntu or Alpine.

But that does not mean it contains a separate Linux kernel.

For example:

```text
Ubuntu container
      ↓
Ubuntu user-space files
      ↓
Host Linux kernel
```

### Important Limitation

Because traditional Linux containers share the host kernel, a Linux container cannot simply bring its own independent Linux kernel the way a VM does.

This is one reason VMs remain useful when stronger isolation or a different kernel/OS is required.

### Interview Point

Don't say:

> "Containers are always more secure than VMs."

Security depends on configuration, kernel isolation, capabilities, privileges, runtime, and workload requirements.

---

## Q9. Can multiple containers be created from the same Docker image?

### Short Interview Answer

**Yes.** A single Docker image can be used to create multiple independent containers.

### Example

Suppose we have:

```text
myapp:v1
```

We can run:

```bash
docker run -d --name app1 myapp:v1
docker run -d --name app2 myapp:v1
docker run -d --name app3 myapp:v1
```

The result is:

```text
             myapp:v1
            /    |    \
           /     |     \
          ▼      ▼      ▼
        app1    app2   app3
```

### Are They Identical?

They start from the same image content, but they are **different runtime instances**.

For example, each can have its own:

- Container name
- IP address
- Process state
- Writable layer
- Environment overrides
- Port mappings
- Mounted volumes

### Why This Is Useful

This is fundamental to horizontal scaling.

For example:

```text
              Load Balancer
                 │
        ┌────────┼────────┐
        ▼        ▼        ▼
      app1      app2     app3
```

All three containers can run the same application image.

### Important Point

The image is shared conceptually and its read-only layers can be reused by the storage system. Docker does not need to create a completely independent copy of all image data for every container.

### Interview Point

This is one of the core reasons container images work well for scalable deployments:

> **Build one image → run many container instances.**

---

## Q10. What happens when a Docker container exits?

### Short Interview Answer

When the container's main process exits, the container normally transitions to the **exited/stopped state**. The container itself still exists until it is removed.

### Example

Suppose:

```dockerfile
CMD ["python", "app.py"]
```

Docker starts the container:

```text
Container
   ↓
python app.py
   ↓
Process exits
   ↓
Container stops
```

You can see the stopped container with:

```bash
docker ps -a
```

### Does the Container Disappear?

No.

This is an important distinction.

After the process exits:

```text
Container
Status = Exited
```

The container metadata and writable layer remain until you remove it.

You can restart it:

```bash
docker start mycontainer
```

Or remove it:

```bash
docker rm mycontainer
```

### What About the Image?

The image is unaffected.

```text
Image
  │
  ├── Container A → Exited
  └── Container B → Running
```

Stopping/removing a container does not automatically remove the image.

### What If You Run `docker run` Again?

Running:

```bash
docker run myimage
```

creates a **new container**.

It does not automatically restart an existing stopped container.

To restart an existing container:

```bash
docker start <container>
```

### Restart Policies

For services, Docker can be configured to restart containers under certain conditions:

```bash
docker run --restart unless-stopped nginx
```

Common policies include:

- `no`
- `on-failure`
- `always`
- `unless-stopped`

### Interview Trap

**Wrong:** "When the application exits, Docker deletes the container."

No. The container normally remains in the exited state unless it is explicitly removed or configured for automatic cleanup, such as using `--rm` for a container.

### Interview Point

Remember:

```text
Process exits
     ↓
Container stops
     ↓
Container still exists
     ↓
docker ps -a
```

---

## Q11. What is the difference between a container and a process?

### Short Interview Answer

A **process** is an executing program instance. A **container** is an isolation and resource-control environment in which one or more processes can run.

### Important Concept

A container is not itself a process in the same sense as a Linux process.

A container is created around processes using kernel isolation and resource-control mechanisms.

Conceptually:

```text
Container
 ├── PID namespace
 ├── Network namespace
 ├── Mount namespace
 ├── Resource controls
 └── Application process
```

### Why Does This Matter?

Suppose:

```bash
docker run nginx
```

The NGINX process runs inside the container's isolated environment.

From inside the container, that process may see a different PID namespace than processes on the host.

For example, the application's process may appear as:

```text
PID 1
```

inside the container.

On the host, the same process has a normal host PID.

So:

```text
Inside container:
PID 1 → nginx

Host:
PID 24573 → nginx
```

The process is still a host-kernel process; the namespace changes its visibility and identity within the container's process namespace.

### Can a Container Have Multiple Processes?

**Yes.**

A container can technically contain multiple processes.

For example:

```text
Container
 ├── Main process
 ├── Child process
 └── Other process
```

However, a common container design principle is to keep **one main application/service responsibility per container**.

This makes lifecycle management, logging, scaling, and health monitoring easier.

### Why Does Docker Care About the Main Process?

Docker associates the container lifecycle with its main process.

If that process exits, the container normally stops.

This is why understanding PID 1 is important for:

- Signal handling
- Graceful shutdown
- Zombie/reaping behavior
- Container lifecycle

### Interview Point

Don't say:

> "A container is a process."

Better:

> **"A container is an isolated runtime environment around processes; the main process determines the container's lifecycle."**

---

## Q12. Is Docker a virtualization technology?

### Short Interview Answer

Docker provides **OS-level/process virtualization or containerization**, but it is fundamentally different from traditional hardware virtualization used by virtual machines.

### Traditional Virtualization

A hypervisor virtualizes hardware resources:

```text
Physical Hardware
       ↓
   Hypervisor
       ↓
 ┌─────┴─────┐
 VM          VM
 │           │
Guest OS    Guest OS
```

Each VM normally has its own guest kernel.

### Docker Containers

Containers generally use the host kernel:

```text
Physical Hardware
       ↓
   Host Kernel
       ↓
 Container Runtime
       ↓
 ┌─────┬─────┬─────┐
 C1    C2    C3
```

Linux namespaces provide isolation, while cgroups provide resource control/accounting.

### Why People Call It "OS-Level Virtualization"

The term comes from the fact that multiple isolated user-space environments can run on one kernel.

But Docker does not emulate an entire physical machine in the way a hypervisor does.

### Important Nuance

Docker Desktop on macOS and Windows uses a Linux virtual machine/VM-backed environment to run Linux containers because the Linux kernel required by Linux containers is not the native host kernel.

So when someone says:

> "Docker doesn't use VMs"

that is too absolute.

A better statement is:

> **"Linux containers themselves are not VMs and normally share a Linux kernel; Docker Desktop may use a VM to provide the Linux environment on non-Linux hosts."**

### Interview Trap

Avoid saying:

> "Docker is completely unrelated to virtualization."

Docker relies on operating-system isolation mechanisms, and Docker Desktop may use virtualization to provide a Linux environment.

### Interview Point

The clean distinction:

```text
VM:
Hardware virtualization
        ↓
Complete guest OS

Container:
OS-level isolation
        ↓
Shared host kernel
```

---

# Quick Revision — Topic 1

| Concept | Remember |
|---|---|
| Docker | Platform for building, packaging, distributing, and running containers |
| Containerization | Packaging/running applications in isolated containers |
| Image | Immutable, layered package/template |
| Container | Runtime instance of an image |
| Image → Container | One image can create many containers |
| Container lifecycle | Main process exits → container normally stops |
| Container vs process | Container provides isolation around processes |
| Containers vs VMs | Containers normally share host kernel; VMs have guest OS/kernel |
| Namespaces | Isolate what processes can see |
| cgroups | Control/account resources |
| Docker problem | Environment consistency and dependency packaging |
| Docker virtualization | OS-level/process isolation, not traditional hardware virtualization |

---

# High-Value Interview Traps

### 1. "Container = VM"

❌ Incorrect.

Containers normally share the host kernel.

### 2. "Image = running container"

❌ Incorrect.

```text
Image → template/package
Container → runtime instance
```

### 3. "Container disappears when its process exits"

❌ Incorrect.

Normally:

```text
Process exits
     ↓
Container = Exited
     ↓
docker ps -a
```

### 4. "One image can only run one container"

❌ Incorrect.

One image can create many independent containers.

### 5. "A container cannot contain multiple processes"

❌ Technically incorrect.

It can, although keeping one main application/service responsibility per container is a common design principle.

### 6. "Docker does not use virtualization"

⚠️ Too broad.

Linux containers are not VMs, but Docker Desktop can use a VM-backed Linux environment.

---

# Topic 1 — Mental Model

The most important model to carry into the next topics is:

```text
                    Docker
                       │
              Builds / Runs
                       │
                       ▼
                 Docker Image
              (immutable layers)
                       │
                  docker run
                       │
                       ▼
                 Docker Container
                       │
          ┌────────────┴────────────┐
          │                         │
     Application                Isolation
       Process               + Resource Control
          │                         │
          └────────────┬────────────┘
                       ▼
                  Host Kernel
```

Once this model is clear, **Docker Architecture, Images, Dockerfile, Networking, Storage, and Container Lifecycle** become much easier to understand.
