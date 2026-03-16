## 🐧 Difference Between `systemd` and `init` (SysVinit)

`init` and `systemd` are **init systems** in Linux. An init system is responsible for **starting, stopping, and managing services during system boot and runtime**.

---

## 🧠 What is `init` (SysVinit)?

📌 `init` is the **traditional and older init system** used in classic Unix and early Linux distributions.

Key points:

* First process started by the kernel (PID 1)
* Uses **runlevels** (0–6)
* Starts services **sequentially**
* Simple but slow

Binary:

```bash
/sbin/init
```

---

## 🧠 What is `systemd`?

📌 `systemd` is a **modern init system and service manager** designed to replace SysVinit.

Key points:

* First process started by the kernel (PID 1)
* Uses **targets** instead of runlevels
* Starts services **in parallel** 🚀
* Provides advanced monitoring and logging

Binary:

```bash
/lib/systemd/systemd
```

---

## 🔁 Boot Flow Comparison (Diagram)

### SysVinit Boot Flow

```text
Kernel
  ↓
init (PID 1)
  ↓
Runlevel scripts (/etc/rc.d/)
  ↓
Services start one by one
```

### systemd Boot Flow

```text
Kernel
  ↓
systemd (PID 1)
  ↓
Targets
  ↓
Services start in parallel (dependency-based)
```

---

## ⚙️ Service Management Comparison

### SysVinit Commands

```bash
service nginx start
service nginx stop
chkconfig nginx on
```

### systemd Commands

```bash
systemctl start nginx
systemctl stop nginx
systemctl enable nginx
```

---

## 📁 Configuration Files

### SysVinit

* Service scripts:

```text
/etc/init.d/nginx
```

* Runlevel directories:

```text
/etc/rc0.d/ to /etc/rc6.d/
```

---

### systemd

* Unit files:

```text
/etc/systemd/system/nginx.service
/usr/lib/systemd/system/nginx.service
```

Example unit file:

```ini
[Unit]
Description=Nginx Web Server
After=network.target

[Service]
ExecStart=/usr/sbin/nginx
Restart=always

[Install]
WantedBy=multi-user.target
```

---

## 🔄 Runlevels vs Targets

### ⚠️ Clarification on Runlevel 2 and 4 (Important)

* **Runlevel 2 and Runlevel 4 exist in SysVinit**, but they are usually **unused or distro-specific**.
* On **RHEL/CentOS/Rocky**, runlevels **2, 3, and 4 behave the same** (multi-user mode).
* **Runlevel 4** is intentionally left **unused for custom purposes** and is rarely used in production.
* In **systemd**, runlevels are kept only for **backward compatibility**.

👉 **Runlevels 2 and 4 are mapped to `multi-user.target`, same as runlevel 3**.

---

### SysVinit Runlevels

### SysVinit Runlevels

| Runlevel | Meaning          |
| -------- | ---------------- |
| 0        | Shutdown         |
| 1        | Single-user      |
| 3        | Multi-user (CLI) |
| 5        | Multi-user (GUI) |
| 6        | Reboot           |

---

### systemd Targets

| Target            | Equivalent |
| ----------------- | ---------- |
| poweroff.target   | Runlevel 0 |
| rescue.target     | Runlevel 1 |
| multi-user.target | Runlevel 3 |
| graphical.target  | Runlevel 5 |
| reboot.target     | Runlevel 6 |

---

## 🚀 Performance & Features Comparison

| Feature             | SysVinit | systemd     |
| ------------------- | -------- | ----------- |
| Boot speed          | Slow     | Fast ⚡      |
| Parallel startup    | ❌        | ✅           |
| Dependency handling | Manual   | Automatic   |
| Logging             | syslog   | journalctl  |
| Resource control    | ❌        | ✅ (cgroups) |
| Service monitoring  | ❌        | ✅           |

---

## 🧪 Real DevOps Example

🔹 Scenario:

* Service fails if network not ready

### SysVinit (Problem)

* Script starts before network
* Causes service failure ❌

### systemd (Solution)

```ini
After=network.target
```

➡️ systemd ensures correct startup order ✅

---

## ⚠️ Common Interview Traps

* systemd is **not just init**, it is a full service manager
* PID 1 is always init system
* systemd replaces cron, syslog, and udev functionality partially

---

## 🎯 Interview One-Liners

* SysVinit starts services sequentially using runlevels
* systemd starts services in parallel using targets and dependencies

---

## 🚀 DevOps Relevance

systemd is critical for:

* Faster boot times
* Reliable service orchestration
* Production troubleshooting (`journalctl`, `systemctl status`)

Understanding this difference is **mandatory for Linux + DevOps interviews** 💡.
