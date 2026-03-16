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
## Ubuntu Boot Process — Detailed Paragraph Summary

When a computer is powered on, the boot process begins with **POST (Power On Self Test)** executed by the system firmware. During this stage, the firmware checks essential hardware components such as the CPU, RAM, keyboard controller, and storage devices to ensure that the hardware is functioning correctly. If any component fails, the system stops the boot process and displays an error or beep code. After POST completes successfully, control moves to the firmware interface, which may be **BIOS (Basic Input Output System)** in legacy systems or **UEFI (Unified Extensible Firmware Interface)** in modern systems. The firmware reads the configured boot order and determines which device (such as SSD, HDD, USB drive, or network device) should be used to start the operating system.

Once the boot device is selected, the firmware locates the bootloader. In legacy BIOS systems, the firmware loads the **MBR (Master Boot Record)**, which is located in the first 512 bytes of the disk. The MBR contains bootloader code that loads the next stage of the bootloader. In UEFI systems, the firmware reads the **EFI System Partition (ESP)** and executes a bootloader file such as ```/boot/efi/EFI/ubuntu/grubx64.efi```. This launches the **GRUB2 (Grand Unified Bootloader version 2)** bootloader. GRUB2 displays the boot menu, allows the user to select a kernel version if multiple kernels are installed, and loads the **Linux kernel image (vmlinuz)** along with the **initramfs image (initrd.img)** into memory.

After GRUB2 loads the kernel, the Linux kernel begins execution. The kernel is responsible for initializing the core components of the operating system including CPU scheduling, memory management, device drivers, interrupt handlers, and hardware communication interfaces. During the early stage of kernel initialization, the system uses **initramfs (Initial RAM Filesystem)**, which is a temporary root filesystem loaded into RAM. This environment contains essential drivers and scripts required to detect hardware devices, initialize storage controllers, assemble RAID or LVM volumes if present, and locate the actual root filesystem stored on disk.

Once the required drivers are loaded and the real root filesystem is located, the system performs a switch_root operation. This replaces the temporary initramfs environment with the real root filesystem mounted from the storage device. After the root filesystem becomes active, the kernel starts the first userspace process, ```/sbin/init```. In modern Linux distributions such as Ubuntu, this process points to systemd. **systemd runs as PID 1** and becomes the main initialization and service management system.

systemd then begins preparing the operating system environment. It mounts additional filesystems, initializes system resources, and determines the system's operational mode by reading the default target defined in ```/etc/systemd/system/default.target```. Depending on the configuration, the system may boot into **multi-user.target (command-line mode) or graphical.target (desktop environment)**. Finally, systemd starts system services based on dependency relationships. Services such as networking, SSH, cron, dbus, and other system components are launched, many of them running in parallel to reduce boot time. Once all required services are started successfully, the system reaches its final state and presents either a login prompt or a graphical desktop, indicating that the operating system is fully booted and ready for use.
