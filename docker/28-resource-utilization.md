# Resource Utilization in Docker

Docker is designed to **run applications efficiently by sharing host resources** instead of virtualizing an entire operating system like traditional VMs. Understanding how Docker uses CPU, memory, disk, and network is critical for performance tuning and cost optimization.

---

## 1. How Docker Uses Host Resources (Big Picture)

Docker containers:

* Run as **Linux processes** on the host
* Share the **same kernel**
* Are isolated using **namespaces**
* Are limited using **cgroups (control groups)**

This means:

* Very low overhead
* Fast startup
* Fine‑grained resource control

---

## 2. CPU Resource Utilization

### How CPU Works in Docker

* Containers share host CPU cores
* Scheduler is the **Linux CFS (Completely Fair Scheduler)**
* CPU usage is controlled using **cgroups**

### CPU Control Options

| Option          | Meaning                         |
| --------------- | ------------------------------- |
| `--cpus`        | Hard limit on CPU cores         |
| `--cpu-shares`  | Relative CPU priority           |
| `--cpuset-cpus` | Pin container to specific cores |

### Example

```bash
docker run -d --name cpu-demo --cpus="1.5" nginx
```

Meaning:

* Container can use **max 1.5 CPU cores**
* If host has 4 cores, container cannot exceed 37.5% total CPU

### CPU Shares Example

```bash
docker run -d --cpu-shares=1024 app1
docker run -d --cpu-shares=512 app2
```

* `app1` gets **2x CPU time** compared to `app2` *during contention*

---

## 3. Memory Resource Utilization

### How Memory Works in Docker

* Containers use host RAM
* No memory is reserved unless explicitly limited
* Controlled using **memory cgroups**

### Memory Control Options

| Option               | Meaning                   |
| -------------------- | ------------------------- |
| `--memory`           | Max RAM container can use |
| `--memory-swap`      | RAM + swap limit          |
| `--oom-kill-disable` | Disable OOM kill          |

### Example

```bash
docker run -d --name mem-demo --memory="512m" --memory-swap="512m" redis
```

Meaning:

* Redis can use **max 512MB RAM**
* No swap allowed
* If exceeded → **OOMKilled**

### What Happens on Memory Exhaustion

1. Container exceeds memory limit
2. Kernel OOM killer triggers
3. Container is terminated
4. Exit code: **137**

---

## 4. Disk (Storage) Resource Utilization

### Storage Drivers

Docker uses **copy-on-write (CoW)** filesystems:

* overlay2 (most common)
* aufs (deprecated)
* devicemapper

### Disk Layers

* Image layers → read-only
* Container layer → writable

### Disk Usage Example

```bash
docker ps -s
```

Output shows:

* Container writable layer size
* Total size including image layers

### Volumes vs Container Layer

| Storage Type    | Performance | Persistence    |
| --------------- | ----------- | -------------- |
| Container layer | Slower      | Lost on delete |
| Volume          | Faster      | Persistent     |
| Bind mount      | Fastest     | Host‑dependent |

---

## 5. Network Resource Utilization

### Networking Model

* Containers use Linux networking stack
* Uses **network namespaces**
* Traffic goes through bridges / overlays

### Common Network Drivers

| Driver  | Use Case         |
| ------- | ---------------- |
| bridge  | Single‑host      |
| host    | High performance |
| overlay | Multi‑host       |

### Network Overhead Example

```bash
docker run --network host nginx
```

* No NAT
* No virtual bridge
* Best network performance

Trade‑off: no isolation

---

## 6. Monitoring Resource Utilization

### docker stats

```bash
docker stats
```

Shows real‑time:

* CPU %
* Memory usage / limit
* Network I/O
* Disk I/O

### Example Output Interpretation

* CPU 120% → using 1.2 cores
* MEM 200MB / 512MB → safe
* NET I/O → traffic per container

---

## 7. Complete Practical Example

### Scenario

Run a web app with controlled resources

```bash
docker run -d \
  --name web-app \
  --cpus="1" \
  --memory="256m" \
  --memory-swap="256m" \
  -p 8080:80 \
  nginx
```

### Result

| Resource | Limit    |
| -------- | -------- |
| CPU      | 1 core   |
| Memory   | 256MB    |
| Swap     | Disabled |
| Network  | Bridge   |

This container:

* Cannot starve host
* Predictable performance
* Safe for multi‑tenant systems

---

## 8. Docker vs VM (Resource Utilization)

| Aspect          | Docker      | VM       |
| --------------- | ----------- | -------- |
| Kernel          | Shared      | Separate |
| Startup         | ms          | minutes  |
| Memory overhead | Low         | High     |
| CPU efficiency  | Near native | Lower    |

---

## 9. Key Takeaways

* Docker uses **Linux kernel features**, not hardware virtualization
* CPU & memory are controlled via **cgroups**
* Isolation is done using **namespaces**
* Resource limits are **mandatory in production**
* Volumes are preferred for I/O heavy workloads

---

If you want next:

* Docker resource limits internals (cgroups v1 vs v2)
* Kubernetes resource requests vs limits mapping
* Real troubleshooting scenarios (CPU throttling, OOMKill)

Tell me what to go deeper into.
