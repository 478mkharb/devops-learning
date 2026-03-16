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
# Netcat and Nmap Commands in Linux (DevOps & Security Use Cases)

## 1. Netcat (nc)

Netcat is a networking utility used for reading and writing data across network connections using TCP or UDP. It is often called the **"Swiss Army knife" of networking**.

### Basic Syntax

```
nc [options] host port
```

### Common Use Cases

#### 1. Check if a Port is Open

```
nc -zv 192.168.1.10 80
```

* `-z` : Scan without sending data
* `-v` : Verbose output

#### 2. Start a Simple TCP Server

```
nc -l 9000
```

* `-l` : Listen mode

#### 3. Send a File Between Machines

Sender:

```
nc -l -p 4444 < file.txt
```

Receiver:

```
nc 192.168.1.20 4444 > file.txt
```

#### 4. Chat Between Two Machines

Server:

```
nc -l 1234
```

Client:

```
nc 192.168.1.20 1234
```

#### 5. Test HTTP Service

```
echo "GET / HTTP/1.0" | nc google.com 80
```

---

## 2. Nmap (Network Mapper)

Nmap is a network scanning and security auditing tool used to discover hosts and services on a network.

### Basic Syntax

```
nmap [scan type] [options] target
```

### Common Use Cases

#### 1. Scan a Single Host

```
nmap 192.168.1.10
```

#### 2. Scan Specific Port

```
nmap -p 22 192.168.1.10
```

#### 3. Scan Multiple Ports

```
nmap -p 22,80,443 192.168.1.10
```

#### 4. Scan Entire Subnet

```
nmap 192.168.1.0/24
```

#### 5. Detect Service Version

```
nmap -sV 192.168.1.10
```

#### 6. OS Detection

```
sudo nmap -O 192.168.1.10
```

#### 7. Aggressive Scan (Most Used in Labs)

```
sudo nmap -A 192.168.1.10
```

This performs:

* OS detection
* Version detection
* Script scanning
* Traceroute

---

## Netcat vs Nmap

| Feature           | Netcat                    | Nmap             |
| ----------------- | ------------------------- | ---------------- |
| Purpose           | Data transfer & debugging | Network scanning |
| Port scanning     | Basic                     | Advanced         |
| Security auditing | Limited                   | Extensive        |
| Server creation   | Yes                       | No               |

---

## DevOps / Security Practical Example

### Step 1: Check open port with Netcat

```
nc -zv 10.0.0.5 22
```

### Step 2: Scan host with Nmap

```
nmap -sV 10.0.0.5
```

### Step 3: Scan full network

```
nmap 10.0.0.0/24
```

This helps identify:

* Running servers
* Open ports
* Vulnerable services

---

## Important Netcat Options

| Option | Meaning                   |
| ------ | ------------------------- |
| -l     | Listen mode               |
| -p     | Port                      |
| -u     | UDP mode                  |
| -v     | Verbose                   |
| -z     | Scan without sending data |

---

## Important Nmap Options

| Option | Meaning            |
| ------ | ------------------ |
| -sS    | TCP SYN scan       |
| -sT    | TCP connect scan   |
| -sU    | UDP scan           |
| -p     | Port specification |
| -A     | Aggressive scan    |
| -O     | OS detection       |

---

## Interview Tip (DevOps / Security)

**Netcat:** Used for debugging network services, testing connectivity, and transferring files.

**Nmap:** Used for network discovery, vulnerability scanning, and security auditing.

## 💡 DevOps Tip

These commands are often the **first 5 minutes of any production outage investigation**.

Mastering them = **faster incident resolution** 🚨.
