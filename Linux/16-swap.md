## 🐧 What is Swap Memory in Linux?

**Swap memory** is a portion of disk space 💾 used by Linux as **virtual memory** when physical RAM is fully or nearly exhausted.

👉 In simple words: **Swap = RAM overflow area on disk** 🧠➡️💽

---

## 🧠 Why Swap Memory is Needed

Linux uses swap to:

* Prevent **Out Of Memory (OOM)** crashes 🚨
* Handle temporary memory spikes
* Improve system stability
* Allow inactive pages to move out of RAM

⚠️ Swap is **slower than RAM**, but better than crashing.

---

## 🔁 How Swap Works (Flow Diagram)

```text
Application needs memory
        ↓
RAM available? ── Yes → Use RAM
        ↓ No
Move inactive pages to Swap
        ↓
RAM freed for active processes
```

---

## 📦 Types of Swap

### 🔹 1. Swap Partition

* Dedicated disk partition
* Better performance
* Fixed size

### 🔹 2. Swap File (Most Common Today)

* Regular file used as swap
* Easy to resize
* Widely used in cloud & DevOps

---

## 🔍 Checking Swap Status

```bash
free -m
swapon --show
```

Example:

```text
NAME      TYPE SIZE USED PRIO
/swapfile file 2G   200M -2
```

---

## 🛠️ Creating Swap File (Step-by-Step)

```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

Verify:

```bash
swapon --show
```

---

## 🧾 Entry in `/etc/fstab`

To make swap **persistent after reboot**, add this line:

```text
/swapfile   none   swap   sw   0   0
```

---

## 🔍 Understanding Each `/etc/fstab` Field

```text
<source>    <mount_point>  <fs_type>  <options>  <dump>  <pass>
```

### 📌 Field-by-field Explanation

### 1️⃣ Source

```text
/swapfile
```

* Swap file or swap partition device

---

### 2️⃣ Mount Point

```text
none
```

* Swap does not mount to a directory

---

### 3️⃣ Filesystem Type

```text
swap
```

* Tells kernel this entry is swap

---

### 4️⃣ Options

```text
sw   (or defaults)
```

* `sw` → enable swap
* `defaults` also works

---

### 5️⃣ Dump Field (0)

```text
0
```

📌 Used by the **dump utility** (backup tool).

* `0` → Do not backup
* Swap should never be dumped

---

### 6️⃣ Pass Field (0)

```text
0
```

📌 Used by **fsck** during boot.

* `0` → Do not check
* Swap is not a filesystem

👉 **This is why swap uses `0 0`** 🎯

---

## 🎯 Why `defaults 0 0` or `sw 0 0`?

* `defaults` / `sw` → enable swap normally
* First `0` → skip dump
* Second `0` → skip fsck

Both are correct and commonly used.

---

## 🧪 Real DevOps Example

🔹 Scenario:

* EC2 instance with 2 GB RAM
* Java app causes OOM crash 🚨

🔹 Fix:

* Add 2 GB swap file
* Persist in `/etc/fstab`

Result:

* No crash
* Slower but stable system ✅

---

## ⚠️ Important Notes (Interview Traps)

* Swap is **not replacement for RAM**
* High swap usage = memory pressure
* Excessive swap can slow system badly

Tune swap behavior:

```bash
cat /proc/sys/vm/swappiness
```

---

## 🎯 Interview One-Liners

**Q: What is swap memory?**
Swap is disk space used as virtual memory when RAM is full.

**Q: Why `0 0` in fstab for swap?**
Swap should not be dumped or fsck-checked.

---

## 🚀 DevOps Relevance

Swap knowledge is critical for:

* Cloud VMs with small RAM ☁️
* Preventing OOM kills
* Production stability

Understanding swap + fstab entries is a **must-have Linux DevOps skill** 💡.
