## 🐧 Essential Linux Commands for DevOps Troubleshooting

These commands are **frequently used in production incidents** to diagnose **performance, disk, memory, and network issues**.

---

## ⏱️ `uptime`

📌 Shows how long the system has been running and the **load average**.

```bash
uptime
```

Example output:

```text
12:40:10 up 15 days,  3 users,  load average: 1.20, 0.95, 0.80
```

Explanation:

* `15 days` → System uptime
* `3 users` → Logged-in users
* Load average → CPU load for 1, 5, 15 minutes

🧠 DevOps use:

* Sudden reboot detection
* CPU load health check

---

## 💾 `df` – Disk Free

📌 Shows disk space usage per filesystem.

```bash
df -h
```

Example:

```text
Filesystem  Size  Used  Avail  Use%  Mounted on
/dev/xvda1   50G   30G   20G   60%   /
```

🧠 DevOps use:

* Detect disk full issues

---

## 📁 `du` – Disk Usage

📌 Shows disk usage of files and directories.

```bash
du -sh /var/log
```

➡️ Helps find **which directory is consuming space**.

---

## 🧾 `df -Th`

📌 Shows disk usage with **filesystem type**.

```bash
df -Th
```

Example:

```text
Filesystem Type Size Used Avail Use% Mounted on
/dev/xvda1 xfs  50G  30G  20G  60% /
```

🧠 DevOps use:

* Know which resize command to use (`xfs_growfs` vs `resize2fs`)

---

## 🧠 `free -m`

📌 Shows memory usage in MB.

```bash
free -m
```

Example:

```text
              total  used  free  buff/cache  available
Mem:          16000  6000  2000     8000       9000
Swap:          2048   100  1948
```

Explanation:

* `buff/cache` → Cached memory (normal)
* `available` → Actual usable memory

🧠 DevOps use:

* Detect memory pressure and OOM risk

---

## 🌐 `netstat` (Legacy but still asked)

📌 Shows network connections, routing tables, ports.

```bash
netstat -tunlp
```

Explanation:

* `t` → TCP
* `u` → UDP
* `n` → Numeric output
* `l` → Listening
* `p` → Process name

⚠️ Deprecated but still common in interviews.

---

## 🌐 `ss -tunlp` (Modern Replacement)

📌 Faster and recommended replacement for `netstat`.

```bash
ss -tunlp
```

Example:

```text
LISTEN 0 128 0.0.0.0:22 users:("sshd",pid=1234)
```

🧠 DevOps use:

* Check which service is listening on which port

---

## 🔒 `chattr` – Change File Attributes

📌 Changes **special file attributes** (beyond permissions).

### Make file immutable (cannot be deleted or modified)

```bash
chattr +i important.conf
```

Verify:

```bash
lsattr important.conf
```

Remove immutability:

```bash
chattr -i important.conf
```

🧠 DevOps use:

* Protect critical config files
* Debug "permission denied" even as root

---

## 🚀 Modern & Very Useful DevOps Commands

### 🔹 `top` / `htop`

* Real-time CPU, memory, process monitoring

### 🔹 `iostat`

```bash
iostat -x
```

* Disk I/O bottleneck analysis

### 🔹 `vmstat`

```bash
vmstat 1
```

* CPU, memory, process, I/O stats

### 🔹 `lsof`

```bash
lsof -i :8080
```

* Find process using a port

### 🔹 `journalctl`

```bash
journalctl -xe
```

* systemd logs

---

## 🧪 Real DevOps Incident Flow

🔍 App slow →

* `uptime` → load high?
* `free -m` → memory pressure?
* `df -h` → disk full?
* `ss -tunlp` → service listening?
* `iostat` → disk wait?

---

## 🎯 Interview One-Liners

* `uptime` → system running time & load
* `df` → disk free
* `du` → directory size
* `free` → memory usage
* `ss` → open ports
* `chattr` → file immutability

---

## 💡 DevOps Tip

These commands are often the **first 5 minutes of any production outage investigation**.

Mastering them = **faster incident resolution** 🚨.
