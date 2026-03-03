## 🐧 Linux Boot Process

The **Linux boot process** describes how a system starts from power-on ⚡ to a fully usable operating system 🖥️.

---

## 🔁 High-Level Boot Flow Diagram

```text
Power ON ⚡
   ↓
BIOS / UEFI 🧠
   ↓
Bootloader (GRUB) 🚀
   ↓
Linux Kernel 🐧
   ↓
Init System (systemd) ⚙️
   ↓
Targets / Services 🧩
   ↓
Login Prompt / Application 🎯
```

---

## 🔹 1. Power On

* System is powered on ⚡
* CPU resets and starts executing firmware code

---

## 🔹 2. BIOS / UEFI

📌 **BIOS (Basic Input Output System)** or **UEFI** performs:

* Power-On Self-Test (POST) 🔍
* Hardware checks (CPU, RAM, Disk)
* Locates the boot device (HDD/SSD/NVMe)

➡️ Hands control to the bootloader

---

## 🔹 3. Bootloader (GRUB)

📌 **GRUB (Grand Unified Bootloader)** loads the Linux kernel.

Responsibilities:

* Displays OS selection menu 🧾
* Loads kernel (`vmlinuz`) into memory
* Loads initial RAM disk (`initramfs`)

Common location:

```bash
/boot/grub2/
```

---

## 🔹 4. Linux Kernel

📌 The kernel is decompressed and initialized 🐧.

Kernel tasks:

* Initializes CPU scheduling 🧠
* Detects and initializes hardware 🔌
* Mounts root filesystem (read-only initially)
* Starts the first user-space process

➡️ Kernel launches **PID 1**

Check kernel version:

```bash
uname -r
```

---

## 🔹 5. Init System (PID 1)

📌 The first process started by the kernel.

Modern Linux uses **systemd** ⚙️ (older: SysVinit, Upstart).

systemd responsibilities:

* Mount filesystems 📂
* Start system services 🧩
* Manage dependencies
* Handle logging and networking

Verify:

```bash
ps -p 1
```

---

## 🔹 6. Targets (Runlevels)

📌 systemd uses **targets** instead of runlevels.

Common targets:

* `multi-user.target` → CLI mode 🖥️
* `graphical.target` → GUI mode 🪟
* `rescue.target` → Single-user mode 🛠️

Check default target:

```bash
systemctl get-default
```

---

## 🔹 7. Services Startup

📌 systemd starts services based on target dependencies.

Examples:

* Network 🌐
* SSH 🔐
* Docker 🐳
* Kubernetes services ☸️

Check service status:

```bash
systemctl status sshd
```

---

## 🔹 8. Login Prompt / Application

📌 System reaches usable state 🎯.

* CLI login → terminal prompt
* GUI login → display manager (gdm, lightdm)

System is now **ready for users and applications** ✅

---

## 🧠 Detailed Internal Flow (Kernel → User Space)

```text
Kernel
  ↓
initramfs
  ↓
Mount root filesystem
  ↓
Start systemd (PID 1)
  ↓
Load targets
  ↓
Start services
  ↓
Login / App
```

---

## 🎯 Interview One-Liner

Linux boot process starts with BIOS/UEFI, loads the bootloader, initializes the kernel, starts systemd (PID 1), loads targets and services, and finally presents the login prompt.

---

## 🚀 DevOps & Troubleshooting Relevance

* Boot stuck at GRUB → Bootloader issue ⚠️
* Boot stuck at initramfs → Disk or UUID issue 💽
* Services not starting → systemd dependency failure 🧩
* Slow boot → Analyze using:

```bash
systemd-analyze blame
```

---

Linux boot process knowledge is critical for **server recovery, cloud VM debugging, and production troubleshooting** 💡.
