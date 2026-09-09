# Docker Interview Preparation — Topic 3: Docker Internals

> **Goal:** Understand what actually happens underneath a Docker container: process isolation, namespaces, cgroups, filesystem layers, copy-on-write, storage drivers, container networking, PID 1, and the relationship between a container and the host kernel.
>
> **Interview focus:** Be able to explain the mechanisms, not just name them.

---

## Q28. How does Docker isolate containers from each other?

### Short Interview Answer

Docker relies primarily on **Linux namespaces** for isolation and **cgroups** for resource control. Additional mechanisms such as Linux capabilities, filesystem isolation, and security profiles can further restrict what a container can do.

### Mental Model

```text
                    Host
                     │
              Linux Kernel
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
    Container A  Container B  Container C
        │            │            │
   namespaces    namespaces    namespaces
   cgroups       cgroups       cgroups
```

### Namespaces

Namespaces provide separate views of system resources.

For example, the PID namespace lets a container have its own process view.

```text
Container A              Container B
PID 1 → app-a             PID 1 → app-b
PID 2 → worker            PID 2 → helper
```

Both containers can have a process appearing as PID 1 inside their own PID namespace.

### cgroups

cgroups control/account resource usage.

For example:

```text
Container A
 ├── CPU limit
 ├── Memory limit
 └── PID limit
```

### Additional Isolation

Docker can also use:

- Linux capabilities
- Seccomp filtering
- AppArmor/SELinux where configured
- Filesystem isolation
- Network namespaces

Therefore, saying only "Docker uses namespaces" is incomplete.

### Interview Point

A strong answer:

> **"Docker primarily uses Linux namespaces for isolation and cgroups for resource control, with capabilities and security mechanisms providing additional restrictions."**

---

## Q29. What is a PID namespace?

### Short Interview Answer

A PID namespace isolates the **process ID view** of a process, allowing processes inside a container to have their own PID numbering and process hierarchy.

### Example

On the host:

```text
Host
 ├── PID 1
 ├── PID 1000
 ├── PID 2000
 └── PID 3000
```

Inside a container:

```text
Container
 ├── PID 1
 ├── PID 2
 └── PID 3
```

The same underlying process can have different PIDs depending on which PID namespace is viewing it.

### Why PID 1 Matters

The first process created in a PID namespace is assigned:

```text
PID 1
```

Inside a normal container, the application's main process often becomes PID 1.

This makes PID 1 special for:

- Signal handling
- Child-process reaping
- Container lifecycle

### Example

```bash
docker run --name web nginx
```

Inside the container, the main NGINX process may appear as:

```text
PID 1
```

On the host, that same process has a different host PID.

### Interview Trap

**Wrong:**

> "PID 1 inside a container is the same process as PID 1 on the host."

No.

Each PID namespace can have its own PID 1.

### Interview Point

```text
Host PID namespace
       │
       └── process → PID 24573

Container PID namespace
       │
       └── same process → PID 1
```

---

## Q30. What is a network namespace?

### Short Interview Answer

A network namespace provides an isolated network view containing its own network interfaces, routes, firewall context, and sockets.

### Conceptual Example

```text
Host Network Namespace
 ├── eth0
 ├── routes
 └── sockets

Container Network Namespace
 ├── eth0
 ├── routes
 └── sockets
```

The container therefore does not simply use the host's exact network namespace when running with normal bridge networking.

### Docker Bridge Networking

A common Docker setup is:

```text
Container
   │
   │ eth0
   ▼
Docker bridge
   │
   ▼
Host network
   │
   ▼
External network
```

The container's network namespace is separate, while Docker provides the connectivity between the container and host/network.

### Why This Matters

It allows two containers to have isolated network stacks.

For example:

```text
Container A → 172.x.x.x
Container B → 172.x.x.x
```

with each address belonging to its own network namespace/network.

### Interview Trap

`localhost` inside a container refers to the container's own network namespace.

So:

```text
Container A
localhost → Container A
```

not:

```text
localhost → Host
```

### Interview Point

Remember:

> **Network namespace = isolated network view.**

---

## Q31. What is a mount namespace?

### Short Interview Answer

A mount namespace isolates the set of filesystem mount points visible to a process.

### Why Docker Uses It

A container needs its own filesystem view.

Conceptually:

```text
Host filesystem
├── /etc
├── /var
├── /home
└── ...

Container filesystem view
├── /etc
├── /var
├── /app
└── ...
```

The container's `/` is different from the host's `/` from the process's point of view.

### Important Point

This does **not** mean that the container has a separate physical disk.

The container filesystem is constructed from image layers and runtime mounts.

### Example

If the container has:

```text
/app
```

that does not mean the process automatically sees the host's `/app`.

Mounts determine what filesystem content is presented inside the container.

### Interview Trap

Do not say:

> "A mount namespace creates a new hard disk."

It creates an isolated **view of mount points**.

### Interview Point

```text
Mount namespace
       ↓
Which filesystem mounts does this process see?
```

---

## Q32. What is a UTS namespace?

### Short Interview Answer

A UTS namespace isolates system identifiers such as the **hostname** and domain name associated with processes.

### Example

Host:

```text
hostname → docker-host
```

Container:

```text
hostname → web-container
```

The container can have its own hostname without changing the host's hostname.

### Why It Matters

Applications sometimes use the hostname for:

- Logging
- Service identification
- Configuration
- Cluster/member identity

The UTS namespace allows the container to have a separate hostname context.

### Interview Point

Think:

```text
UTS namespace
      ↓
Hostname/domain identity
```

---

## Q33. What is an IPC namespace?

### Short Interview Answer

An IPC namespace isolates certain **inter-process communication resources**, such as System V IPC objects and POSIX message queues.

### Why It Matters

Processes in different IPC namespaces can be isolated from one another's IPC resources.

Conceptually:

```text
Container A
 IPC resources A

Container B
 IPC resources B
```

This helps prevent unrelated workloads from freely sharing the same IPC namespace.

### Interview Point

You generally do not need to memorize every IPC mechanism for a basic Docker interview.

Know:

> **IPC namespace isolates IPC resources between process groups.**

---

## Q34. What is a user namespace?

### Short Interview Answer

A user namespace isolates user and group ID mappings so that a process can have different UID/GID identities inside the namespace than it has on the host.

### Why This Matters

User namespaces can reduce the impact of running processes with high privileges inside a container.

Conceptually:

```text
Inside container:
UID 0 → root

Host:
UID 100000 → mapped identity
```

The exact mapping depends on configuration.

### Important Distinction

Without user namespace remapping, container root may correspond to host root from the kernel's UID perspective.

That is one reason running containers as non-root and reducing privileges is important.

### Interview Point

User namespaces are an additional isolation/security mechanism.

They are different from:

```text
PID namespace → process identity/view
NET namespace → network view
USER namespace → user/group identity mapping
```

---

## Q35. What is copy-on-write in Docker?

### Short Interview Answer

Copy-on-write (CoW) allows containers to share read-only image layers and create a private writable copy of data only when modifications require it, depending on the storage driver.

### Layer Model

```text
             Container
                 │
          Writable Layer
                 │
       ───────────────────
        Read-only Layer 3
       ───────────────────
        Read-only Layer 2
       ───────────────────
        Read-only Layer 1
```

Multiple containers can share the same read-only image layers.

### Example

Suppose two containers use:

```text
ubuntu:24.04
```

They can share the underlying image layers.

If Container A modifies a file from an image layer, the storage system can make the changed version available in the container's writable layer rather than modifying the shared image layer.

Conceptually:

```text
Image layer
   │
   ├── Container A
   │      └── modified copy
   │
   └── Container B
          └── original view
```

### Why This Is Useful

CoW helps reduce:

- Duplicate storage
- Image startup/storage overhead
- Unnecessary copying of unchanged files

### Important Nuance

The exact behavior depends on the storage driver and filesystem implementation.

Do not claim that every Docker storage driver implements CoW in exactly the same way.

### Interview Point

The key idea:

> **Read-only image data is shared; modifications are isolated in the container's writable storage layer.**

---

## Q36. What is the container writable layer?

### Short Interview Answer

The container writable layer is the runtime layer where changes made to the container's filesystem are stored when they are not written to a mounted volume or bind mount.

### Example

Image:

```text
/app/app.py
```

Container modifies:

```text
/app/config.json
```

The modification is stored in the container's writable layer under the storage driver's model.

### Layer Structure

```text
Container writable layer
─────────────────────────
Image layer 3
─────────────────────────
Image layer 2
─────────────────────────
Image layer 1
```

### What Happens When the Container Is Removed?

Normally:

```bash
docker rm mycontainer
```

removes the container and its writable layer.

Therefore, data stored only there should not be treated as persistent application data.

### Persistent Data

Use:

```text
Docker volume
or
Bind mount
```

for data that must survive container replacement/removal.

### Interview Trap

**Wrong:**

> "The container writes directly into the image."

No.

The image layers remain read-only.

### Interview Point

```text
Image → read-only
Container → writable layer
Volume → persistent storage
```

---

## Q37. What are Docker storage drivers?

### Short Interview Answer

Docker storage drivers implement how image layers and container writable layers are stored and combined on the host.

### Why Do They Exist?

Docker needs to represent:

```text
Image layers
       +
Container writable layer
```

as a usable container filesystem.

The storage driver handles this according to its implementation.

### Common Examples

Depending on platform and configuration, Docker can use drivers such as:

- `overlay2`
- `fuse-overlayfs`
- `btrfs`
- `zfs`

The available/default driver depends on the host environment.

### `overlay2`

On many modern Linux Docker installations, `overlay2` is commonly used.

Conceptually:

```text
Container writable layer
          ↓
     overlay filesystem
          ↓
   Read-only image layers
```

### Important Nuance

Do not say:

> "`overlay2` is Docker itself."

It is a storage driver used by Docker to implement the layered filesystem behavior.

### How to Check

```bash
docker info
```

Look for the storage driver information.

### Interview Point

Think:

> **Storage driver = implementation of the container/image filesystem layer model.**

---

## Q38. What is the difference between image layers and the container writable layer?

### Short Interview Answer

Image layers are normally **read-only and shared**, while the container writable layer stores runtime filesystem changes specific to that container.

### Comparison

| Image Layers | Container Writable Layer |
|---|---|
| Read-only | Writable |
| Part of image | Belongs to container |
| Can be shared | Specific to container |
| Immutable image content | Runtime changes |
| Persist as part of image | Removed with container unless data is mounted elsewhere |

### Example

```text
Image
 ├── Ubuntu layer
 ├── Python layer
 └── Application layer
          ↓
     Read-only
          ↓
Container
 └── Writable layer
```

### Important Point

If ten containers use the same image:

```text
              Same image
           / / / | \ \ \
          C1 C2 C3 C4 C5 ...
```

they do not each need a separate complete copy of every unchanged image layer.

### Interview Point

This distinction is foundational to understanding Docker image storage efficiency.

---

## Q39. Why should application data not normally be stored in the container writable layer?

### Short Interview Answer

Because the container writable layer is tied to the container lifecycle and is not designed to be the primary mechanism for durable application data.

### Example

Suppose a PostgreSQL container stores database files only inside its writable layer.

If the container is removed:

```text
Container removed
       ↓
Writable layer removed
       ↓
Database data lost
```

This is undesirable.

### Better Approach

Use persistent storage:

```text
Container
   │
   ├── Application filesystem
   │
   └── /data
        │
        ▼
      Volume
```

### Why Volumes?

Volumes are managed separately from the container lifecycle.

```text
Container A ──┐
               ├── Docker Volume
Container B ──┘
```

A replacement container can mount the same volume.

### Interview Point

Containers should generally be treated as **replaceable/ephemeral compute**, while persistent application data should live outside the container writable layer.

---

## Q40. What happens to files written inside a container when the container is deleted?

### Short Interview Answer

Files written only to the container's writable layer are normally deleted with the container. Files stored in a volume or bind mount can survive container deletion.

### Example

```bash
docker run --name test ubuntu
```

Suppose the container writes:

```text
/tmp/data.txt
```

That file is in the container's writable filesystem unless `/tmp` is mounted elsewhere.

If you remove:

```bash
docker rm test
```

the container-specific writable data is removed.

### Volume Example

```bash
docker volume create appdata

docker run \
  --mount source=appdata,target=/data \
  myapp
```

Data written to:

```text
/data
```

is stored in the volume.

Removing the container does not automatically remove the volume.

### Interview Trap

**Wrong:**

> "All container data disappears when the container stops."

Stopping is different from removing.

```text
docker stop
    ↓
Container remains
    ↓
Writable layer remains
```

Whereas:

```text
docker rm
    ↓
Container removed
    ↓
Container writable layer removed
```

### Interview Point

Always distinguish:

```text
stop ≠ remove
```

---

## Q41. What is PID 1 inside a Docker container?

### Short Interview Answer

PID 1 is the first process in the container's PID namespace and normally serves as the container's main process. The container's lifecycle is tied to this main process.

### Example

```bash
docker run python:3.12 python app.py
```

Conceptually:

```text
Container PID namespace

PID 1
 └── python app.py
```

If PID 1 exits:

```text
PID 1 exits
     ↓
Container exits
```

### Why PID 1 Is Special

PID 1 has special responsibilities in Linux process management.

Two interview-relevant issues are:

1. **Signal handling**
2. **Child-process reaping**

### Signal Handling

Docker can request graceful termination of a container.

For example:

```bash
docker stop myapp
```

The container's main process needs to handle the termination signal appropriately for graceful shutdown.

### Child Reaping

If an application creates child processes and does not correctly manage them, PID 1 responsibilities become important for reaping orphaned/zombie processes.

### Init Process

For workloads where the application is not suitable as PID 1, Docker can use an init process:

```bash
docker run --init myapp
```

This adds an init process to help handle signal forwarding and child reaping.

### Interview Point

The critical concept:

> **The main container process is PID 1 in the container's PID namespace, and its exit normally causes the container to stop.**

---

## Q42. Why is PID 1 signal handling important in Docker?

### Short Interview Answer

Because Docker's graceful-stop behavior relies on signals reaching the container's main process, and PID 1 has special signal semantics in Linux.

### Example

When you run:

```bash
docker stop myapp
```

Docker requests graceful termination of the container.

Conceptually:

```text
docker stop
     ↓
Container runtime
     ↓
PID 1
     ↓
Application handles termination
     ↓
Graceful shutdown
```

If the application handles the termination signal correctly, it can:

- Stop accepting new requests
- Finish existing work
- Close connections
- Flush buffers
- Exit cleanly

### Shell Wrapper Problem

Consider:

```dockerfile
CMD ["./start.sh"]
```

If `start.sh` launches another process incorrectly and does not use proper `exec`, the intended application may not be PID 1.

A common pattern is:

```bash
exec python app.py
```

The `exec` replaces the shell process with the application process.

### Better Exec-Form Example

```dockerfile
CMD ["python", "app.py"]
```

This avoids an unnecessary shell wrapper.

### Interview Trap

Don't say:

> "Shell form never receives signals."

The more precise explanation is:

> **Exec form avoids an intermediate shell, allowing the intended application to run directly as PID 1 and making signal behavior more predictable.**

### Interview Point

PID 1 + signals + graceful shutdown is a common Docker interview topic.

---

## Q43. What happens when PID 1 inside a container exits?

### Short Interview Answer

When the container's main process, normally PID 1 in its PID namespace, exits, Docker normally considers the container stopped.

### Example

```text
Container
   │
   └── PID 1 → application
                  │
                  ▼
               exits
                  │
                  ▼
            Container exits
```

### Important Distinction

The container itself does not necessarily get deleted.

You can see it using:

```bash
docker ps -a
```

It may show:

```text
Exited (0)
```

or another exit code.

### Restart Policy

If a restart policy is configured, Docker may restart the container depending on the policy and exit condition.

For example:

```bash
docker run --restart on-failure myapp
```

### Interview Point

Remember:

```text
PID 1 exits
     ↓
Container stops
     ↓
Restart policy may act
```

---

## Q44. What is the difference between `docker stop` and `docker kill` internally?

### Short Interview Answer

`docker stop` requests a **graceful shutdown**, while `docker kill` sends a termination signal directly without the normal graceful-stop sequence.

### `docker stop`

Conceptually:

```text
docker stop
     ↓
termination signal
     ↓
application gets time to shut down
     ↓
if it does not exit within the configured timeout
     ↓
forced termination
```

The exact signal and timeout behavior can be configured.

### `docker kill`

By default:

```bash
docker kill myapp
```

sends:

```text
SIGKILL
```

which cannot be caught or handled by the application.

### Comparison

| Command | Purpose |
|---|---|
| `docker stop` | Graceful shutdown |
| `docker kill` | Immediate/forced termination |
| `docker restart` | Stop + start behavior |

### Why It Matters

Production applications should generally support graceful shutdown.

For example, a web application should have time to:

```text
Stop accepting new requests
        ↓
Finish in-flight requests
        ↓
Close DB connections
        ↓
Flush logs
        ↓
Exit
```

### Interview Point

```text
stop → graceful
kill → forceful
```

---

## Q45. What is an exit code in Docker?

### Short Interview Answer

A container exit code is the status returned by its main process when it terminates.

### Common Examples

```text
Exit code 0
    ↓
Normal/successful completion

Non-zero
    ↓
Application/runtime error or another failure condition
```

For example:

```bash
docker ps -a
```

may show:

```text
Exited (0)
Exited (1)
Exited (137)
```

### Exit Code 137

A commonly asked interview question.

137 is:

```text
128 + 9
```

where:

```text
9 = SIGKILL
```

Therefore, 137 indicates the process was terminated by SIGKILL.

### Is 137 Always OOM?

**No.**

OOM killing is a common reason, but exit code 137 alone does not prove that the kernel OOM killer terminated the process.

Investigate:

```bash
docker inspect <container>
docker stats
dmesg
journalctl
```

and the host/container runtime logs as appropriate.

### Interview Trap

❌ "Exit 137 always means out of memory."

Better:

> **"Exit 137 means termination by SIGKILL; OOM is a common cause, but you should verify the actual cause."**

---

## Q46. How does Docker networking work internally at a high level?

### Short Interview Answer

With normal bridge networking, Docker gives the container its own network namespace and virtual network interface, connects that interface to a Docker-managed bridge, and uses host networking mechanisms to provide connectivity beyond the container.

### High-Level Flow

```text
Container Network Namespace
          │
       eth0/veth
          │
          ▼
   Docker Bridge
          │
          ▼
       Host
          │
          ▼
   External Network
```

### veth Pair

A common Linux mechanism is a **veth pair**.

Conceptually:

```text
Container namespace
      eth0
       │
       │ veth pair
       │
       ▼
Host namespace
      veth
       │
       ▼
Docker bridge
```

One end is in the container's network namespace; the other end is attached to the host-side networking setup.

### Container IP

Docker can assign the container an IP on the Docker network.

For example:

```text
Container A → 172.x.x.x
Container B → 172.x.x.x
```

### External Connectivity

Depending on the network configuration, Docker can use Linux networking/NAT mechanisms to allow traffic between container networks and external networks.

### Interview Point

For an interview, focus on:

```text
Network namespace
      ↓
veth pair
      ↓
Docker bridge
      ↓
Host networking/NAT
```

rather than memorizing implementation details that vary by network mode/platform.

---

## Q47. What is a virtual Ethernet pair (veth pair)?

### Short Interview Answer

A veth pair is a pair of interconnected virtual network interfaces commonly used to connect a container's network namespace to networking resources in the host namespace.

### Conceptual Diagram

```text
Container Namespace
┌─────────────────────┐
│       eth0          │
└──────────┬──────────┘
           │
           │ veth pair
           │
┌──────────▼──────────┐
│    Host-side veth   │
└──────────┬──────────┘
           │
           ▼
     Docker bridge
```

Packets entering one end emerge from the other.

### Why Docker Uses It

The container needs a network interface while remaining in its own network namespace.

The veth pair creates a virtual connection between:

```text
Container network namespace
          ↕
Host network namespace
```

### Interview Point

A useful one-line answer:

> **"A veth pair acts like a virtual network cable connecting two network namespaces."**

That is a mental model, not a literal physical cable.

---

## Q48. What is Docker bridge networking?

### Short Interview Answer

Bridge networking connects containers to a Linux bridge on the host, allowing containers on the same Docker network to communicate while remaining in separate network namespaces.

### Conceptual Architecture

```text
        Container A
             │
            eth0
             │
             ▼
      ┌─────────────┐
      │ Docker      │
      │ bridge      │
      └──────┬──────┘
             │
             ▼
        Container B
```

For external access:

```text
Containers
    ↓
Docker bridge
    ↓
Host networking
    ↓
External network
```

### Default vs User-Defined Bridge

Docker has a default `bridge` network, but for application stacks, a **user-defined bridge network** is generally preferred.

Why?

User-defined bridge networks provide better:

- Isolation
- Name-based service discovery
- Network configuration control

### Example

```bash
docker network create app-net

docker run -d --name db --network app-net postgres
docker run -d --name api --network app-net myapi
```

The API container can communicate with the database using the Docker network's name-based discovery:

```text
db:5432
```

### Interview Trap

Do not confuse:

```text
Container port
Host published port
```

A container can communicate internally without publishing its port to the host.

### Interview Point

```text
User-defined bridge
        ↓
Container-to-container communication
        ↓
Service/container name discovery
```

---

## Q49. What does `localhost` mean inside a container?

### Short Interview Answer

`localhost` inside a container refers to the **container's own network namespace**, not the Docker host and not another container.

### Example

Suppose:

```text
Container A
  API → localhost:5432
```

This means:

```text
Container A → Container A
```

It does **not** mean:

```text
Container A → Host
```

and it does not mean:

```text
Container A → Database Container
```

### Correct Container-to-Container Communication

If:

```text
API container
DB container
```

are on the same user-defined network:

```text
API → db:5432
```

rather than:

```text
API → localhost:5432
```

### Why This Is a Common Bug

Developers often move an application from a host-based environment into containers and leave:

```text
DB_HOST=localhost
```

The application then tries to find the database inside its own container.

### Interview Point

Memorize:

> **Inside a container, localhost means "this container."**

---

## Q50. What happens when a container connects to another container using its name?

### Short Interview Answer

On a user-defined Docker network, Docker provides service/container name-based DNS resolution so that a container can resolve another container's name to its network address.

### Example

```bash
docker network create app-net

docker run -d --name db --network app-net postgres
docker run -d --name api --network app-net myapi
```

The API can use:

```text
db:5432
```

Docker's embedded DNS resolves:

```text
db
 ↓
Database container's network IP
```

### Why This Is Better Than Hardcoding IPs

Container IPs can change when containers are recreated.

For example:

```text
Old DB container
172.x.x.5

Recreated DB container
172.x.x.8
```

The name can remain:

```text
db
```

so the application configuration does not need to track the changing IP.

### Interview Point

This is the key Docker networking principle:

> **Use stable names, not container IP addresses, for application-to-application communication.**

---

## Q51. How does Docker provide resource isolation?

### Short Interview Answer

Docker uses Linux cgroups to control and account for resource usage, while namespaces isolate the process environment.

### Example

Suppose:

```bash
docker run \
  --memory=512m \
  --cpus=1 \
  myapp
```

Conceptually:

```text
myapp container
      │
      ├── Namespace isolation
      │
      └── cgroup
            ├── memory limit
            └── CPU limit
```

### Why Resource Limits Matter

Without appropriate controls, a CPU- or memory-intensive workload can affect other workloads on the same host.

### CPU

Docker can apply CPU-related controls.

### Memory

Docker can apply memory limits.

### PIDs

Docker can also limit the number of processes:

```bash
docker run --pids-limit=100 myapp
```

### Important Nuance

A resource limit is not necessarily the same thing as guaranteed performance.

For example:

```text
--cpus=1
```

does not mean the application will always receive one full CPU under all host conditions.

It defines a resource control boundary according to the underlying cgroup configuration.

### Interview Point

```text
Namespaces → isolation
cgroups    → resource control/accounting
```

---

## Q52. What is the difference between isolation and resource limiting in Docker?

### Short Interview Answer

**Isolation** determines what resources/processes a container can see, while **resource limiting** controls how much CPU, memory, PIDs, and other resources its processes can consume.

### Comparison

| Isolation | Resource Control |
|---|---|
| Namespaces | cgroups |
| Process visibility | CPU |
| Network view | Memory |
| Filesystem/mount view | PIDs |
| Hostname/user identity | Other supported resource controls |

### Example

```text
Container A
 ├── PID namespace
 ├── Network namespace
 └── Mount namespace

Container A
 └── cgroup
      ├── CPU limit
      └── memory limit
```

### Interview Trap

A container having its own IP does not mean it has its own kernel.

Likewise, a memory limit does not mean the container has a separate memory device.

### Interview Point

This distinction is one of the most reusable Docker concepts:

> **Namespaces answer "what can I see?" while cgroups answer "how much can I use?"**

---

## Q53. What is the Docker container filesystem?

### Short Interview Answer

A container's filesystem is normally constructed from read-only image layers plus a container-specific writable layer, with additional mounts such as volumes or bind mounts optionally overlaid at runtime.

### Conceptual View

```text
Container filesystem
        │
        ├── Writable layer
        │
        ├── Image layer 3
        ├── Image layer 2
        └── Image layer 1
                 +
        Runtime mounts
        ├── Volume
        └── Bind mount
```

### Why This Matters

The path:

```text
/app/data
```

could come from different underlying storage depending on whether it is:

- Part of the image
- In the writable layer
- A volume
- A bind mount

### Example

```bash
docker run \
  --mount source=mydata,target=/data \
  myapp
```

The `/data` path inside the container is backed by the volume rather than ordinary container writable-layer storage.

### Interview Point

Do not think:

> "A container is just a folder on the host."

Its filesystem view is assembled through multiple kernel/storage mechanisms.

---

## Q54. What is the relationship between a Docker image, storage driver, and container filesystem?

### Short Interview Answer

The image provides immutable filesystem layers, the storage driver manages how those layers are stored and combined, and the container adds a writable runtime layer on top.

### Flow

```text
Docker Image
 ├── Layer 1
 ├── Layer 2
 └── Layer 3
       │
       ▼
Storage Driver
       │
       ▼
Container Filesystem
       │
       └── Writable Layer
```

### Example

When an image is built:

```text
Dockerfile
   ↓
Image layers
```

When a container starts:

```text
Image layers
    +
Writable layer
    ↓
Container filesystem
```

### Why This Matters

It explains:

- Why images can be reused
- Why containers can have isolated filesystem changes
- Why deleting a container does not necessarily delete the image
- Why volumes are used for persistence

### Interview Point

Keep these concepts separate:

```text
Image       → content/package
Storage     → how filesystem layers are represented
Container   → runtime filesystem + process environment
Volume      → persistent storage
```

---

## Q55. What are Linux capabilities in Docker?

### Short Interview Answer

Linux capabilities divide traditional root privileges into smaller privilege units, allowing Docker to grant or remove specific kernel-level privileges instead of giving every container process unrestricted root capabilities.

### Why Capabilities Matter

Traditional Unix root has broad privileges.

Linux capabilities split many privileged operations into separate units.

Conceptually:

```text
Full root privileges
       ↓
Capabilities
 ├── CAP_NET_ADMIN
 ├── CAP_SYS_ADMIN
 ├── CAP_CHOWN
 └── ...
```

Docker can drop unnecessary capabilities.

### Security Principle

Prefer:

```text
Minimum required privileges
```

rather than:

```text
All privileges
```

### Example Concept

A web application may not need capabilities required for low-level network administration.

Removing unnecessary capabilities reduces attack surface.

### Interview Trap

**Wrong:**

> "Running as root means the container automatically has every possible host privilege."

Container privileges are affected by namespaces, capabilities, seccomp, security profiles, runtime configuration, and other controls.

### Interview Point

Capabilities are one layer of container security, not the entire security model.

---

## Q56. What is seccomp in Docker?

### Short Interview Answer

Seccomp, or secure computing mode, can restrict the system calls a container process is allowed to make.

### Why System Calls Matter

Applications interact with the Linux kernel through system calls:

```text
Application
     ↓
system call
     ↓
Linux kernel
```

A container does not need every possible system call to perform its job.

A seccomp profile can restrict which calls are permitted.

### Security Model

```text
Container Process
       │
       ▼
    seccomp
       │
       ▼
Linux system calls
       │
       ▼
Linux kernel
```

### Why This Helps

If an application is compromised, restricting unnecessary system calls can reduce the attack surface available to the attacker.

### Important Nuance

Seccomp is not a replacement for:

- Namespaces
- cgroups
- Capabilities
- AppArmor/SELinux
- Non-root execution
- Secure image practices

It is one layer in defense-in-depth.

### Interview Point

Remember:

> **Seccomp restricts system calls; capabilities restrict specific privileged operations.**

---

## Q57. Why should containers generally run as a non-root user?

### Short Interview Answer

Running as a non-root user reduces the privileges available to the application inside the container and can reduce the impact of a container compromise.

### Example Dockerfile

```dockerfile
FROM python:3.12

RUN useradd --create-home appuser

WORKDIR /app
COPY . /app

USER appuser

CMD ["python", "app.py"]
```

The application then runs as:

```text
appuser
```

rather than root.

### Why This Is Better

If the application is compromised:

```text
Root process
    ↓
Greater potential privileges

Non-root process
    ↓
Reduced privileges
```

### Important Nuance

Running as non-root is not a complete security solution.

Other controls still matter:

- Namespaces
- Capabilities
- Seccomp
- AppArmor/SELinux
- Read-only filesystems
- Image security
- Network controls

### Interview Point

Use the principle:

> **Run with the minimum privileges required.**

---

## Q58. What does `--privileged` do, and why is it dangerous?

### Short Interview Answer

`--privileged` substantially relaxes container isolation and grants the container many additional Linux capabilities/devices/permissions. It should not be used as a routine fix for permission errors.

### Why It Is Dangerous

Normal containers run with restrictions.

`--privileged` removes or relaxes many of those restrictions.

Conceptually:

```text
Normal container
    ↓
Restricted capabilities/devices

--privileged
    ↓
Much broader access
```

This can significantly increase the impact of a container compromise.

### Common Bad Practice

Developer encounters:

```text
Permission denied
```

and changes:

```bash
docker run --privileged ...
```

This may make the error disappear, but it can create a serious security problem.

### Better Approach

Determine what permission is actually required.

For example:

- Add a specific capability
- Change ownership
- Run as an appropriate user
- Mount only the required device
- Adjust the security profile
- Use the appropriate volume permissions

### Interview Point

A strong answer:

> **"`--privileged` is a broad security relaxation, not a normal permission-fix switch."**

---

# Topic 3 — Deep Mental Model

The complete model from this topic is:

```text
                    Docker Container
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
      Processes         Filesystem        Network
          │                │                │
          │                │                │
   PID namespace     Image layers       Network namespace
   USER namespace    + writable layer   + veth
                     + mounts           + bridge
          │                │                │
          └────────────────┼────────────────┘
                           │
                           ▼
                     Linux Kernel
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
          namespaces      cgroups   capabilities
                                        │
                                     seccomp
```

---

# The 5 Most Important Internal Concepts

## 1. Namespace

```text
"What can this process see?"
```

Examples:

- Processes
- Network
- Mounts
- Hostname
- Users

## 2. cgroup

```text
"How much can this process use?"
```

Examples:

- CPU
- Memory
- PIDs

## 3. Image Layers

```text
"Where does the container's initial filesystem come from?"
```

Read-only layers form the image.

## 4. Writable Layer

```text
"Where do runtime filesystem changes go?"
```

Container-specific writable storage.

## 5. PID 1

```text
"Which process controls the container lifecycle?"
```

The main process in the container's PID namespace.

---

# High-Value Interview Traps

### Trap 1 — Container = VM

❌ No.

A Linux container normally shares the host kernel.

### Trap 2 — Namespace = resource limit

❌ No.

```text
Namespace → isolation
cgroup    → resource control
```

### Trap 3 — `localhost` = Docker host

❌ No.

Inside a container:

```text
localhost → same container
```

### Trap 4 — Container stop deletes data

❌ No.

```text
stop → container remains
rm   → container removed
```

### Trap 5 — Container writable layer = persistent storage

❌ No.

Use volumes/bind mounts for data that must survive container replacement/removal.

### Trap 6 — Exit 137 always means OOM

❌ No.

```text
137 = 128 + SIGKILL(9)
```

OOM is a common cause, not proof.

### Trap 7 — `--privileged` fixes permissions safely

❌ No.

It substantially weakens isolation.

### Trap 8 — Root inside container is always harmless

❌ No.

Container root has important privileges; reduce privileges where practical.

### Trap 9 — PID 1 is just another process

⚠️ Not quite.

PID 1 has special Linux semantics and is important for signal handling and child reaping.

---

# Interview Follow-Up Questions

An interviewer can move from this topic into:

1. What are Linux namespaces?
2. What is a PID namespace?
3. What is a network namespace?
4. What is a mount namespace?
5. What are cgroups?
6. What is copy-on-write?
7. What is `overlay2`?
8. What is the container writable layer?
9. Why do containers need volumes?
10. What is PID 1?
11. Why does PID 1 matter?
12. What happens when PID 1 exits?
13. Why does `docker stop` behave differently from `docker kill`?
14. What does exit code 137 mean?
15. How does Docker networking work internally?
16. What is a veth pair?
17. What does `localhost` mean inside a container?
18. What are Linux capabilities?
19. What is seccomp?
20. Why should containers run as non-root?
21. What does `--privileged` do?
22. How does Docker isolate containers?
23. How does Docker limit CPU and memory?
24. What happens to container data when the container is removed?

---

# Final Interview Answer — "Explain Docker Internals"

If the interviewer asks:

> **"Explain how Docker provides container isolation internally."**

A strong answer is:

> "Docker containers are processes running on the host kernel, not separate virtual machines. Docker uses Linux namespaces to isolate the process, network, mount, hostname, IPC, and user views. It uses cgroups to control and account for resources such as CPU, memory, and process count. The container filesystem is built from read-only image layers plus a container-specific writable layer, with volumes or bind mounts used for persistent data. Additional security controls such as Linux capabilities, seccomp, and security profiles can further restrict the container. The container's main process normally runs as PID 1 in its PID namespace, and its exit normally causes the container to stop."

That answer demonstrates **architecture + Linux internals + filesystem + lifecycle + security** rather than just definitions.
