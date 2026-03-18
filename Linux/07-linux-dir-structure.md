## 🐧 Linux File System Structure (Directory Hierarchy)

Linux follows a **single-rooted directory structure** 🌳 where everything starts from the **root directory `/`**. Each directory has a **specific purpose**, especially important for **system administration and DevOps**.

---

## 🌳 Linux Directory Structure (Tree View)

```text
/
├── bin        → Essential user commands
├── sbin       → System/admin commands
├── boot       → Kernel & bootloader files
├── dev        → Device files
├── etc        → System configuration files
├── home       → User home directories
│   └── user
├── root       → Root user home directory
├── lib        → Essential shared libraries
├── lib64      → 64-bit libraries
├── usr        → User System Resources
│   ├── bin
│   ├── sbin
│   └── lib
├── var        → Variable data (logs, spool)
│   ├── log
│   └── tmp
├── tmp        → Temporary files
├── proc       → Process & kernel info (virtual)
├── sys        → Kernel & hardware info
├── mnt        → Temporary mount point
├── media      → Removable media mounts
├── opt        → Optional / third-party software
├── run        → Runtime process data
└── srv        → Service data
```

---

## 🌳 Root Directory `/`

📌 The top-level directory of Linux.

* All files, directories, and devices originate here
* Only critical system directories exist directly under `/`

---

## 📁 `/bin` – Essential User Binaries

📌 Contains **basic command binaries** required for all users.

Examples:

* `ls`, `cp`, `mv`, `cat`, `bash`

🧠 Needed for system to work in **single-user or rescue mode**.

---

## 📁 `/sbin` – System Binaries

📌 Contains **administrative commands** mainly for root user.

Examples:

* `ip`, `reboot`, `fsck`, `mount`

🛠️ Used for system maintenance and recovery.

---

## 📁 `/boot` – Boot Loader Files

📌 Stores files required during **system boot**.

Contains:

* Linux kernel (`vmlinuz`)
* Initramfs (`initramfs.img`)
* GRUB configuration

⚠️ Critical directory – accidental deletion may break boot.

---

## 📁 `/dev` – Device Files

📌 Represents hardware devices as files 🔌.

Examples:

* `/dev/sda` → Disk
* `/dev/null` → Discards output
* `/dev/random` → Random data

🧠 Used by kernel and applications to interact with hardware.

---

## 📁 `/etc` – Configuration Files

📌 Contains **system-wide configuration files**.

Examples:

* `/etc/passwd` → User accounts
* `/etc/shadow` → Passwords
* `/etc/fstab` → Mount points
* `/etc/ssh/sshd_config`

🛠️ No binaries here, only configs.

---

## 📁 `/home` – User Home Directories

📌 Personal directories for normal users 👤.

Example:

* `/home/mukesh`

Contains:

* User files
* Hidden configs (`.bashrc`, `.ssh`)

---

## 📁 `/root` – Root User Home

📌 Home directory for **root user** only.

⚠️ Different from `/`.

---

## 📁 `/lib` & `/lib64` – Shared Libraries

📌 Contains essential shared libraries needed by `/bin` and `/sbin`.

Examples:

* `libc.so`
* Kernel modules

🧠 Required for system startup.

---

## 📁 `/usr` – User System Resources

📌 Contains **non-essential system binaries and libraries**.

Subdirectories:

* `/usr/bin` → User commands
* `/usr/sbin` → Admin commands
* `/usr/lib` → Libraries

📌 Installed applications usually go here.

---

## 📁 `/var` – Variable Data

📌 Stores data that **changes frequently** 🔄.

Examples:

* `/var/log` → Logs 📜
* `/var/spool` → Mail, cron jobs
* `/var/tmp` → Temp files (persistent)

🚨 Log files growing here can fill disk.

---

## 📁 `/tmp` – Temporary Files

📌 Stores short-lived temporary files 🧪.

* Cleared on reboot (usually)
* World-writable

⚠️ Do not store important data.

---

## 📁 `/proc` – Process Information (Virtual FS)

📌 Virtual filesystem exposing **kernel and process info** 🧠.

Examples:

* `/proc/cpuinfo`
* `/proc/meminfo`
* `/proc/<pid>`

🧠 No real files stored on disk.

---

## 📁 `/sys` – System Information

📌 Interface between kernel and hardware.

Used for:

* Hardware info
* Kernel tuning

Example:

* `/sys/class/net`

---

## 📁 `/mnt` – Temporary Mount Point

📌 Used to temporarily mount filesystems.

Example:

```bash
mount /dev/sdb1 /mnt
```

---

## 📁 `/media` – Removable Media

📌 Auto-mounted removable devices 💽.

Examples:

* USB drives
* CD/DVD

---

## 📁 `/opt` – Optional Software

📌 Used for **third-party or custom applications**.

Examples:

* `/opt/oracle`
* `/opt/jenkins`

---

## 📁 `/run` – Runtime Data

📌 Stores runtime information since last boot.

Examples:

* PID files
* Sockets

🧠 Replaces older `/var/run`.

---

## 📁 `/srv` – Service Data

📌 Data served by services.

Examples:

* Web content
* FTP data

---

## 🎯 Interview One-Liner

Linux uses a hierarchical file system starting from `/`, where each directory has a predefined purpose for binaries, configuration, logs, users, devices, and runtime data.

---

## 🚀 DevOps Relevance

Understanding Linux directory structure helps in:

* Faster troubleshooting ⚡
* Disk usage analysis 📊
* Secure configuration management 🔐
* Production incident handling 🛠️
