## 🐧 What is an Inode in Linux?

An **inode (index node)** is a **data structure** used by Linux filesystems to store **metadata about a file or directory**, but **not the file name or actual data**.

👉 In simple words: **inode = identity card of a file** 🪪

---

## 🧠 What Information Does an Inode Store?

Each inode contains:

* 📌 File type (file, directory, link)
* 📏 File size
* 👤 Owner (UID)
* 👥 Group (GID)
* 🔐 File permissions (rwx)
* ⏰ Timestamps (access, modify, change)
* 🔢 Number of hard links
* 📍 Pointers to data blocks on disk

❌ Inode does NOT store:

* File name
* File content

---

## 🔗 Relationship Between Filename and Inode

* A **filename** is just a human-readable label 🏷️
* Directory maps **filename → inode number**
* Multiple filenames can point to the **same inode** (hard links)

---

## 🔍 Viewing Inode Number

```bash
ls -i file.txt
```

Example output:

```text
123456 file.txt
```

➡️ `123456` is the inode number

---

## 📊 Checking Inode Details

```bash
stat file.txt
```

Shows:

* Inode number
* Size
* Permissions
* Owner
* Timestamps

---

## 🧩 Inode and Hard Links (Important)

```bash
ln file.txt file_hardlink.txt
```

Check:

```bash
ls -li
```

Example:

```text
123456 -rw-r--r-- 2 user user file.txt
123456 -rw-r--r-- 2 user user file_hardlink.txt
```

➡️ Same inode, link count = 2
➡️ Deleting one name does NOT delete data

---

## 🔗 Inode and Soft Links

```bash
ln -s file.txt file_symlink.txt
```

* Symlink has **different inode**
* Points to filename, not inode

If original file is deleted → symlink breaks ❌

---

## 💾 Inode Allocation on Disk

* Inodes are created when filesystem is created
* Total number of inodes is **fixed**

Check inode usage:

```bash
df -i
```

Example:

```text
Filesystem  Inodes  IUsed  IFree  IUse%  Mounted on
/dev/xvda1  655360  20000  635360  4%     /
```

---

## 🚨 Inode Exhaustion (Very Important)

📌 Disk can have **free space but no free inodes** ❗

Common causes:

* Millions of small files
* Logs, cache, temp files

Symptoms:

* Cannot create new files
* Error: `No space left on device`

Fix:

* Delete unused small files
* Clean logs/cache

---

## 🧪 Real DevOps Example

🔹 Scenario:

* `df -h` shows 40% disk free
* App still fails to write files 🚨

🔹 Check:

```bash
df -i
```

➡️ Inodes exhausted
➡️ Remove unused files from `/var/log`, `/tmp`

---

## 📊 Inode vs File (Quick Comparison)

| Aspect            | Inode | File Name |
| ----------------- | ----- | --------- |
| Stores metadata   | ✅     | ❌         |
| Stores data       | ❌     | ❌         |
| Unique identifier | ✅     | ❌         |
| Human readable    | ❌     | ✅         |

---

## 🎯 Interview One-Liners

**Q: What is an inode?**
An inode is a filesystem data structure that stores metadata and disk block pointers for a file.

**Q: Why disk full but space available?**
Because inodes are exhausted.

---

## 🚀 DevOps Relevance

Understanding inodes helps in:

* Disk full incidents 🔥
* Log explosion troubleshooting 📜
* Performance tuning
* Production stability

Inode knowledge is a **must-have Linux interview topic** 💡.
