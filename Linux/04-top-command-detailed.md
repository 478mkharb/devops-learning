## 🐧 `top` Command in Linux

The **`top` command** is a real-time system monitoring tool that displays **CPU, memory usage, load average, and running processes**. It is widely used by **Linux administrators and DevOps engineers** to troubleshoot performance issues.

---

## ▶️ How to Run `top`

```bash
top
```

The output is divided into **two main sections**:

1. **System summary (upper section)**
2. **Process list (lower section)**

---

## 🔝 System Summary Section (Top Area)

### ⏱️ 1. Uptime & System Info

Example:

```text
top - 12:10:45 up 10 days,  3:21,  2 users,  load average: 0.45, 0.60, 0.75
```

**Explanation:**

* `12:10:45` → Current system time
* `up 10 days, 3:21` → System uptime
* `2 users` → Logged-in users

---

### 📊 2. Load Average (VERY IMPORTANT)

```text
load average: 0.45, 0.60, 0.75
```

**Meaning:**

* `0.45` → Average load over last **1 minute**
* `0.60` → Average load over last **5 minutes**
* `0.75` → Average load over last **15 minutes**

**Interpretation:**

* Load ≈ number of CPU cores → Normal
* Load > CPU cores → CPU bottleneck ⚠️

📌 Example:

* 4-core system with load `8.0` → Overloaded CPU

---

### 🧮 3. Tasks (Process Summary)

```text
Tasks: 210 total, 2 running, 208 sleeping, 0 stopped, 0 zombie
```

**Explanation:**

* `total` → Total processes
* `running` → Currently using CPU
* `sleeping` → Waiting for resources
* `stopped` → Paused processes
* `zombie` → Terminated but not cleaned 🧟

---

### 🔥 4. CPU Usage Breakdown

```text
%Cpu(s): 10.5 us,  2.0 sy,  0.0 ni, 85.0 id,  1.0 wa,  0.3 hi,  0.2 si,  0.0 st
```

**Each field explained:**

* `us` → User space CPU usage (applications)
* `sy` → System/kernel CPU usage
* `ni` → Nice (low priority processes)
* `id` → Idle CPU (higher is good)
* `wa` → I/O wait (disk or network delay ⚠️)
* `hi` → Hardware interrupts
* `si` → Software interrupts
* `st` → Steal time (virtual machines)

📌 Example:

* High `wa` → Disk or NFS issue
* High `sy` → Kernel or driver overhead

---

### 🧠 5. Memory (RAM) Usage

```text
MiB Mem : 16000 total, 4000 free, 9000 used, 3000 buff/cache
```

**Explanation:**

* `total` → Total RAM
* `free` → Unused memory
* `used` → Used by processes
* `buff/cache` → Cached data for performance

📌 Important:

* High `buff/cache` is **normal** and improves speed

---

### 💾 6. Swap Memory

```text
MiB Swap: 2048 total, 1800 free, 248 used
```

**Explanation:**

* Swap is disk-based virtual memory
* Used when RAM is exhausted

📌 Example:

* High swap usage → Memory pressure ⚠️

---

## 📋 Process List Section (Bottom Area)

### 🧾 Common Columns

| Column  | Meaning           |
| ------- | ----------------- |
| PID     | Process ID        |
| USER    | Owner of process  |
| PR      | Priority          |
| NI      | Nice value        |
| VIRT    | Virtual memory    |
| RES     | Physical RAM used |
| SHR     | Shared memory     |
| S       | Process state     |
| %CPU    | CPU usage         |
| %MEM    | Memory usage      |
| TIME+   | CPU time          |
| COMMAND | Process name      |

---

## 🧪 Example Use Case (DevOps)

🔍 Scenario: Application is slow

* Load average high → CPU issue
* `wa` high → Disk/NFS bottleneck
* High `%MEM` → Memory leak
* Zombie processes → App bug

---

## ⌨️ Useful `top` Shortcuts

* `P` → Sort by CPU usage
* `M` → Sort by memory
* `k` → Kill a process
* `1` → Show per-CPU usage
* `q` → Quit

---

## 🎯 Interview One-Liner

The `top` command provides real-time visibility into system performance, showing CPU, memory, load average, and running processes to help diagnose Linux performance issues.

---

## 🚀 DevOps Tip

In production, always correlate:

* **Load average + CPU cores**
* **High wa + disk latency**
* **Swap usage + OOM risk**

This makes `top` one of the most important Linux troubleshooting tools 💡.
