# Container Lifecycle in Docker

A Docker container has a **well-defined lifecycle**, starting from an image and ending with container removal. Understanding this lifecycle is critical for debugging crashes, restarts, OOMKills, health checks, and orchestration behavior (Docker / Kubernetes).

---

## 1. High-Level Container Lifecycle

A container moves through these main phases:

```
Image → Created → Running → (Paused) → Stopped → Removed
```

Not all containers pass through every state (e.g., pause is optional).

---

## 2. Full Container Lifecycle Flow (End-to-End)

```
┌──────────┐
│ Docker   │
│  Image   │
└────┬─────┘
     │ docker run
     ▼
┌──────────┐
│ Created  │  ← container metadata created
│          │  ← writable layer created
└────┬─────┘
     │ start
     ▼
┌──────────┐
│ Running  │  ← PID 1 started
│          │  ← namespaces + cgroups active
└────┬─────┘
     │ stop / crash / OOM
     ▼
┌──────────┐
│ Stopped  │  ← process exited
│          │  ← resources released
└────┬─────┘
     │ docker rm
     ▼
┌──────────┐
│ Removed  │  ← metadata + writable layer deleted
└──────────┘
```

---

## 3. Step-by-Step Lifecycle (Deep Dive)

### Step 1: Image Exists

* Image is built or pulled (`docker build`, `docker pull`)
* Image contains:

  * Read-only layers
  * Entrypoint / CMD
  * Metadata

Image **does not consume CPU or memory** until container starts.

---

### Step 2: Container Creation (Created State)

Command:

```bash
docker create nginx
```

What happens internally:

1. Docker creates **container metadata**
2. Creates a **writable container layer**
3. Assigns:

   * Container ID
   * Network namespace
   * Cgroup paths
4. ENTRYPOINT / CMD resolved

Important:

* No process running yet
* No CPU or memory usage

State:

```
STATUS: Created
```

---

### Step 3: Container Start (Running State)

Command:

```bash
docker start <container>
```

or directly:

```bash
docker run nginx
```

What happens internally:

1. Linux namespaces created:

   * PID
   * Network
   * Mount
   * IPC
   * UTS
2. Cgroups applied:

   * CPU limits
   * Memory limits
3. `runc` is invoked
4. PID 1 process starts
5. STDOUT/STDERR attached

State:

```
STATUS: Running
```

Key rule:

> **If PID 1 exits, container exits**

---

### Step 4: Running Phase (Active Lifecycle)

While running, container can:

* Serve requests
* Write logs
* Consume CPU & memory
* Access volumes

You can:

```bash
docker logs <container>
docker exec -it <container> bash
docker stats
```

Optional transitions:

* Health check failures
* Restart policies

---

### Step 5: Paused State (Optional)

Command:

```bash
docker pause <container>
```

What happens:

* All processes receive **SIGSTOP**
* CPU usage becomes 0
* Memory remains allocated

Resume:

```bash
docker unpause <container>
```

---

### Step 6: Container Stop (Stopped / Exited State)

Ways a container stops:

1. Manual stop:

```bash
docker stop <container>
```

2. Application crash
3. OOM Kill
4. SIGTERM / SIGKILL

Internal flow:

```
SIGTERM → (grace period) → SIGKILL
```

State:

```
STATUS: Exited
```

Exit codes:

* `0` → normal exit
* `137` → OOMKilled / SIGKILL
* `1` → application error

---

### Step 7: Restart (If Policy Exists)

Restart policies:

```bash
--restart always
--restart on-failure
--restart unless-stopped
```

Lifecycle with restart:

```
Exited → Restarting → Running
```

Docker does **not recreate the container**, it restarts the same one.

---

### Step 8: Container Removal (Removed State)

Command:

```bash
docker rm <container>
```

What gets deleted:

* Container metadata
* Writable layer
* Network namespace

What stays:

* Image
* Volumes (unless `-v` used)

Final state:

```
STATUS: Removed
```

---

## 4. Complete Lifecycle with Commands

| Stage   | Command               |
| ------- | --------------------- |
| Image   | `docker pull nginx`   |
| Create  | `docker create nginx` |
| Start   | `docker start <id>`   |
| Run     | `docker run nginx`    |
| Pause   | `docker pause <id>`   |
| Stop    | `docker stop <id>`    |
| Restart | `docker restart <id>` |
| Remove  | `docker rm <id>`      |

---

## 5. Real Example: Nginx Container Lifecycle

```bash
docker run -d --name web nginx
```

Flow:

1. Image pulled (if not present)
2. Container created
3. PID 1 = nginx master process
4. Container running

Now stop it:

```bash
docker stop web
```

* nginx gets SIGTERM
* exits gracefully
* container moves to Exited state

Remove it:

```bash
docker rm web
```

Lifecycle complete.

---

## 6. Docker vs Kubernetes Lifecycle (Quick Mapping)

| Docker         | Kubernetes        |
| -------------- | ----------------- |
| Image          | Image             |
| Container      | Pod               |
| Restart policy | Pod restartPolicy |
| docker stop    | kubelet SIGTERM   |
| docker rm      | Pod deletion      |

---

## 7. Key Takeaways

* Container = **process with isolation**
* PID 1 controls container lifetime
* Image is immutable, container is mutable
* Stopped containers still consume disk
* Volumes outlive containers
* Restart ≠ recreate

---

If you want next:

* Kubernetes Pod lifecycle (with init & sidecars)
* PID 1 signal handling best practices
* Why containers exit immediately
* Container crash & restart troubleshooting

Tell me what to continue with.
