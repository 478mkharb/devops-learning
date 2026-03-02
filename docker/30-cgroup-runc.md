# cgroups and runc — Deep Dive with Full Flow Diagrams

This document explains **cgroups** and **runc** from first principles and shows **how they work together** to create and control a container. This is the exact mental model used by the Linux kernel, Docker, and Kubernetes.

---

## 1. Why cgroups and runc exist (the real problem)

Before containers:

* Processes competed freely for CPU & memory
* One bad process could crash the entire host
* No clean standard to create isolated processes

Linux solved this in two layers:

1. **Namespaces** → isolation (who you are, what you see)
2. **cgroups** → limits & accounting (how much you can use)

`runc` is the tool that **creates the process using both**.

---

## 2. What is cgroups?

**cgroups (Control Groups)** are a Linux kernel feature that:

* Limits resource usage
* Measures resource usage
* Enforces fairness

> cgroups do NOT create containers. They only **control resources**.

---

## 3. What resources cgroups control

| Resource  | What kernel enforces    |
| --------- | ----------------------- |
| CPU       | Scheduling time         |
| Memory    | Hard memory limits      |
| PIDs      | Max number of processes |
| Block I/O | Disk throughput         |

---

## 4. What is runc?

**`runc` is a low-level OCI runtime**.

Its job:

* Create namespaces
* Apply cgroups
* Start the container process (PID 1)

> After starting the process, `runc` exits.

---

## 5. High-Level Relationship (must remember)

```
Namespaces → Isolation
cgroups    → Limits
runc       → Creates the container process
Kernel     → Enforces everything
```

---

## 6. Full Container Creation Flow (Docker example)

```
docker run nginx
   ↓
Docker CLI
   ↓
Docker Daemon
   ↓
containerd
   ↓
OCI Bundle (rootfs + config.json)
   ↓
runc create
   ↓
┌───────────────────────────┐
│ Linux Kernel              │
│ • namespaces created      │
│ • cgroups applied         │
│ • rootfs mounted          │
└───────────────────────────┘
   ↓
runc start
   ↓
PID 1 starts (nginx)
   ↓
runc exits
```

Container keeps running **without runc**.

---

## 7. cgroups in action (CPU example)

### Command

```bash
docker run --cpus=1 nginx
```

### Kernel flow

```
Container process
   ↓
CPU cgroup
   ↓
CFS Scheduler
   ↓
Physical CPU
```

If container exceeds CPU:

* Kernel **throttles CPU time**
* Process is NOT killed

---

## 8. cgroups in action (Memory example)

### Command

```bash
docker run --memory=256m redis
```

### Kernel flow

```
Process allocates memory
   ↓
Memory cgroup checks limit
   ↓
Limit exceeded
   ↓
OOM Killer triggered
   ↓
Process killed (exit code 137)
```

Memory limits are **hard limits**.

---

## 9. cgroups filesystem (where limits live)

Modern systems (cgroup v2):

```bash
/sys/fs/cgroup/
  └── docker/
      └── <container-id>/
          ├── cpu.max
          ├── memory.max
          ├── pids.max
```

These files are written by Docker/containerd and enforced by the kernel.

---

## 10. runc lifecycle (very important)

```
runc create
   ↓
Sets up container (paused state)
   ↓
runc start
   ↓
Starts PID 1
   ↓
runc exits
```

runc is **not a daemon**.

---

## 11. PID 1 rule (container golden rule)

```
PID 1 exits
   ↓
Container exits
```

This is why:

* Bad signal handling breaks containers
* Shell scripts as PID 1 cause issues

---

## 12. Restart flow with cgroups & runc

```
Container crashes
   ↓
Docker detects exit
   ↓
containerd prepares bundle again
   ↓
runc start (same container)
   ↓
PID 1 restarted
```

cgroups are **reused**, not recreated.

---

## 13. Kubernetes flow (same internals)

```
Pod Spec (requests & limits)
   ↓
kubelet
   ↓
CRI (containerd)
   ↓
runc
   ↓
Namespaces + cgroups
   ↓
Container process
```

Kubernetes does NOT replace cgroups or runc — it **uses them**.

---

## 14. cgroups vs namespaces vs runc (table)

| Component | Purpose                  |
| --------- | ------------------------ |
| Namespace | Isolation                |
| cgroup    | Resource limits          |
| runc      | Create container process |
| Kernel    | Enforcement              |

---

## 15. One-page mental model (interview ready)

```
Image
  ↓
OCI bundle
  ↓
runc
  ↓
(namespaces + cgroups)
  ↓
PID 1
  ↓
Linux kernel enforces limits
```

---

## 16. Key Takeaways

* Containers are **Linux processes**
* cgroups limit and measure resources
* runc creates the container process
* Kernel enforces everything
* Docker/Kubernetes only orchestrate

---

Next possible deep dives:

* cgroups v1 vs v2 (with real files)
* Mapping Kubernetes limits → cgroup values
* CPU throttling vs starvation
* Debugging OOMKilled containers

Tell me what to add next.
