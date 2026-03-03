## 💽 What is RAID?

**RAID (Redundant Array of Independent Disks)** is a technique that combines multiple physical disks into a **single logical unit** to improve:

* ⚡ Performance
* 🔐 Fault tolerance (data protection)
* 📈 Storage capacity

RAID is commonly used in **servers, databases, and production systems**.

---

## 🧠 Why RAID is Used

Without RAID:

* Single disk failure = data loss ❌

With RAID:

* Data can be **mirrored, striped, or parity-protected** ✅

---

## 🔁 How RAID Works (Concept)

```text
Application
   ↓
Operating System
   ↓
RAID Layer
   ↓
Multiple Physical Disks
```

---

## 📊 RAID 0 – Striping (Performance Only)

### 📌 What is RAID 0?

RAID 0 splits (stripes) data **across multiple disks** to improve performance.

* ❌ No redundancy
* ❌ No fault tolerance
* ✅ High performance

---

### 🧩 How RAID 0 Works (Example)

```text
Disk 1: A1   A3   A5
Disk 2: A2   A4   A6
```

➡️ Data is written alternately across disks

---

### 📈 Characteristics

* Minimum disks: **2**
* Usable storage: **100%**
* Disk failure tolerance: **0 disks**

---

### 🧪 Example Use Case

* Cache servers
* Temporary data
* High-speed workloads

⚠️ **If one disk fails → all data is lost**

---

## 🪞 RAID 1 – Mirroring (High Availability)

### 📌 What is RAID 1?

RAID 1 creates an **exact copy (mirror)** of data on multiple disks.

* ✅ High fault tolerance
* ❌ Lower usable storage
* ✅ Easy recovery

---

### 🧩 How RAID 1 Works (Example)

```text
Disk 1: A B C D
Disk 2: A B C D
```

➡️ Same data written to both disks

---

### 📈 Characteristics

* Minimum disks: **2**
* Usable storage: **50%**
* Disk failure tolerance: **1 disk**

---

### 🧪 Example Use Case

* OS disks
* Critical system partitions
* Small databases

---

## 🧮 RAID 5 – Striping with Parity (Balanced)

### 📌 What is RAID 5?

RAID 5 uses **striping + distributed parity** to provide **fault tolerance with better storage efficiency**.

* ✅ Fault tolerant
* ✅ Efficient storage
* ⚠️ Slower writes (parity calculation)

---

### 🧩 How RAID 5 Works (Example)

```text
Disk 1: A1   A4   P3
Disk 2: A2   P2   A6
Disk 3: P1   A5   A7
```

* `A` → Data block
* `P` → Parity block

Parity rotates across disks 🔄

---

### 📈 Characteristics

* Minimum disks: **3**
* Usable storage: **(N−1) disks**
* Disk failure tolerance: **1 disk**

---

### 🧪 Example Use Case

* File servers
* Application servers
* Read-heavy workloads

---

## 📊 RAID 0 vs RAID 1 vs RAID 5 (Comparison Table)

| Feature         | RAID 0   | RAID 1    | RAID 5            |
| --------------- | -------- | --------- | ----------------- |
| Technique       | Striping | Mirroring | Striping + Parity |
| Min Disks       | 2        | 2         | 3                 |
| Usable Space    | 100%     | 50%       | (N−1)/N           |
| Fault Tolerance | ❌ None   | ✅ 1 disk  | ✅ 1 disk          |
| Performance     | ⭐⭐⭐⭐     | ⭐⭐⭐       | ⭐⭐                |
| Cost            | Low      | High      | Medium            |

---

## 🚨 RAID is NOT Backup (Interview Trap)

RAID protects against:

* Disk failure ✅

RAID does NOT protect against:

* File deletion ❌
* Corruption ❌
* Ransomware ❌

👉 **Always combine RAID with backups**

---

## 🎯 Interview One-Liners

* **RAID 0** → Performance, no redundancy
* **RAID 1** → Mirroring, high availability
* **RAID 5** → Balanced performance and fault tolerance

---

## 🚀 DevOps Relevance

RAID knowledge helps in:

* Server design 🏗️
* Disk failure handling
* Capacity planning
* Production reliability

RAID questions are **very common in Linux & DevOps interviews** 💡.
