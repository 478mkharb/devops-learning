## 🐧 Linux Special Permissions: SUID, SGID & Sticky Bit

Linux provides **special permission bits** that go beyond normal `rwx` permissions. These are critical for **security, shared directories, and controlled privilege escalation**.

Special permissions:

* 🔑 **SUID (Set User ID)**
* 👥 **SGID (Set Group ID)**
* 📌 **Sticky Bit**

---

## 🔑 1. SUID (Set User ID)

### 📌 What is SUID?

SUID allows a **user to execute a file with the permissions of the file owner**, not the user who runs it.

👉 Mostly used when normal users need **temporary elevated privileges**.

---

### ⚙️ How SUID Works (Concept)

```text
User runs command
        ↓
Command executes as file OWNER (usually root)
```

---

### 🔍 Identifying SUID

```bash
ls -l /usr/bin/passwd
```

Example:

```text
-rwsr-xr-x 1 root root /usr/bin/passwd
```

* `s` in **owner execute position** → SUID enabled

---

### 🧪 Practical Example (Very Important)

`passwd` command:

* Updates `/etc/shadow` (owned by root)
* Normal users cannot edit it ❌
* SUID allows temporary root privilege ✅

---

### ➕ Setting & Removing SUID

```bash
chmod u+s file
chmod u-s file
```

Numeric form:

```bash
chmod 4755 file
```

---

### ⚠️ Security Note (Interview Trap)

* SUID on scripts is ignored on most systems
* Misconfigured SUID binaries = privilege escalation risk 🚨

---

## 👥 2. SGID (Set Group ID)

### 📌 What is SGID?

SGID causes a file or directory to be executed or created with the **group ownership of the parent**, not the user’s default group.

---

### 📁 SGID on Directories (MOST COMMON)

📌 Files created inside inherit the **same group**.

---

### 🧪 Practical Example

```bash
mkdir /shared
chown :devops /shared
chmod 2775 /shared
```

Now:

* Any file created inside `/shared`
* Will belong to group `devops`

Check:

```bash
ls -ld /shared
```

Example output:

```text
drwxrwsr-x 2 root devops /shared
```

* `s` in **group execute position** → SGID enabled

---

### ➕ Setting & Removing SGID

```bash
chmod g+s dir
chmod g-s dir
```

Numeric form:

```bash
chmod 2755 dir
```

---

## 📌 3. Sticky Bit

### 📌 What is Sticky Bit?

Sticky bit ensures that **only the file owner can delete or rename their files**, even if the directory is writable by everyone.

---

### 🧪 Classic Example: `/tmp`

```bash
ls -ld /tmp
```

Example:

```text
drwxrwxrwt 10 root root /tmp
```

* `t` in **others execute position** → Sticky bit enabled

---

### 🔐 Why Sticky Bit is Needed

Without sticky bit:

* Any user could delete others’ files ❌

With sticky bit:

* Only owner or root can delete files ✅

---

### ➕ Setting & Removing Sticky Bit

```bash
chmod +t dir
chmod -t dir
```

Numeric form:

```bash
chmod 1777 dir
```

---

## 🧠 Visual Permission Positions

```text
-rwsrwsrwt
 │ │ │ │
 │ │ │ └── Sticky Bit
 │ │ └──── SGID
 │ └────── SUID
 └──────── File type
```

---

## 📊 Comparison Table

| Permission | Applies To | Purpose           | Common Example |
| ---------- | ---------- | ----------------- | -------------- |
| SUID       | File       | Run as owner      | passwd         |
| SGID       | Dir/File   | Group inheritance | shared dirs    |
| Sticky     | Directory  | Safe deletion     | /tmp           |

---

## 🎯 Interview One-Liners

* **SUID** → Execute file as owner
* **SGID** → Inherit group ownership
* **Sticky Bit** → Protect files in shared directories

---

## 🚀 DevOps Relevance

These permissions are critical for:

* Secure privilege escalation 🔐
* Shared CI/CD workspaces 🤝
* Multi-user servers

Misuse can lead to **major security vulnerabilities**, so DevOps engineers must understand them deeply 💡.
