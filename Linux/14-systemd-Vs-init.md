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

## Ubuntu Boot Process — Detailed Paragraph Summary

When a computer is powered on, the boot process begins with **POST (Power On Self Test)** executed by the system firmware. During this stage, the firmware checks essential hardware components such as the CPU, RAM, keyboard controller, and storage devices to ensure that the hardware is functioning correctly. If any component fails, the system stops the boot process and displays an error or beep code. After POST completes successfully, control moves to the firmware interface, which may be **BIOS (Basic Input Output System)** in legacy systems or **UEFI (Unified Extensible Firmware Interface)** in modern systems. The firmware reads the configured boot order and determines which device (such as SSD, HDD, USB drive, or network device) should be used to start the operating system.

Once the boot device is selected, the firmware locates the bootloader. In legacy BIOS systems, the firmware loads the **MBR (Master Boot Record)**, which is located in the first 512 bytes of the disk. The MBR contains bootloader code that loads the next stage of the bootloader. In UEFI systems, the firmware reads the **EFI System Partition (ESP)** and executes a bootloader file such as ```/boot/efi/EFI/ubuntu/grubx64.efi```. This launches the **GRUB2 (Grand Unified Bootloader version 2)** bootloader. GRUB2 displays the boot menu, allows the user to select a kernel version if multiple kernels are installed, and loads the **Linux kernel image (vmlinuz)** along with the **initramfs image (initrd.img)** into memory.

After GRUB2 loads the kernel, the Linux kernel begins execution. The kernel is responsible for initializing the core components of the operating system including CPU scheduling, memory management, device drivers, interrupt handlers, and hardware communication interfaces. During the early stage of kernel initialization, the system uses **initramfs (Initial RAM Filesystem)**, which is a temporary root filesystem loaded into RAM. This environment contains essential drivers and scripts required to detect hardware devices, initialize storage controllers, assemble RAID or LVM volumes if present, and locate the actual root filesystem stored on disk.

Once the required drivers are loaded and the real root filesystem is located, the system performs a switch_root operation. This replaces the temporary initramfs environment with the real root filesystem mounted from the storage device. After the root filesystem becomes active, the kernel starts the first userspace process, ```/sbin/init```. In modern Linux distributions such as Ubuntu, this process points to systemd. **systemd runs as PID 1** and becomes the main initialization and service management system.

systemd then begins preparing the operating system environment. It mounts additional filesystems, initializes system resources, and determines the system's operational mode by reading the default target defined in ```/etc/systemd/system/default.target```. Depending on the configuration, the system may boot into **multi-user.target (command-line mode) or graphical.target (desktop environment)**. Finally, systemd starts system services based on dependency relationships. Services such as networking, SSH, cron, dbus, and other system components are launched, many of them running in parallel to reduce boot time. Once all required services are started successfully, the system reaches its final state and presents either a login prompt or a graphical desktop, indicating that the operating system is fully booted and ready for use.
