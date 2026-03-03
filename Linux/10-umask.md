## 🐧 What is `umask` in Linux?

`umask` (**user file-creation mode mask**) defines the **default permissions** that are **removed** when a new **file or directory** is created 🧠.

👉 In simple words: **umask decides what permissions NOT to give by default** ❌🔐

---

## 🧠 Why umask Exists

* Prevents files from being created with unsafe permissions 🚫
* Enforces security by default 🔐
* Controls permissions **before** `chmod` is applied

---

## 🔢 Default Permission Rules (VERY IMPORTANT)

Linux starts with **base permissions**:

* 📄 **Files** → `666` (`rw-rw-rw-`)
* 📁 **Directories** → `777` (`rwxrwxrwx`)

👉 `umask` is **subtracted** from these defaults.

---

## ➖ Permission Formula (Interview Gold)

```text
Final Permission = Base Permission − umask
```

---

## 🎯 Example 1: umask = 022 (Most Common)

### 📄 Files

```text
Base:   666
umask:  022
----------------
Result: 644 (rw-r--r--)
```

### 📁 Directories

```text
Base:   777
umask:  022
----------------
Result: 755 (rwxr-xr-x)
```

➡️ Owner full access, group & others read-only 👤✅

---

## 🎯 Example 2: umask = 027 (More Secure)

### 📄 Files

```text
Base:   666
umask:  027
----------------
Result: 640 (rw-r-----)
```

### 📁 Directories

```text
Base:   777
umask:  027
----------------
Result: 750 (rwxr-x---)
```

➡️ Used on production servers 🔐

---

## 🔍 Checking Current umask

```bash
umask
```

Example output:

```text
0022
```

---

## 🔁 Setting umask Temporarily

```bash
umask 027
```

➡️ Applies only to current shell session ⏳

---

## 🔒 Setting umask Permanently

For a user:

```bash
~/.bashrc
~/.profile
```

For system-wide:

```bash
/etc/profile
/etc/bashrc
```

---

## 🧪 Practical DevOps Example

🔹 Scenario:

* App creates log files
* Logs should NOT be world-readable 🚨

🔹 Solution:

```bash
umask 027
```

➡️ Logs created as `640`
➡️ Secure by default 🛡️

---

## ⚠️ Common Mistakes

* Expecting execute bit on files ❌
* Forgetting umask affects **new files only**
* Using very open umask like `000` 🚫

---

## 📊 umask Quick Reference

| umask | File Perm | Dir Perm | Use Case         |
| ----- | --------- | -------- | ---------------- |
| 022   | 644       | 755      | Default Linux    |
| 027   | 640       | 750      | Secure servers   |
| 077   | 600       | 700      | Highly sensitive |

---

## 🎯 Interview One-Liners

**Q: What is umask?**
`umask` defines default permission bits that are removed during file or directory creation.

**Q: Default umask value?**
Usually `022`.

---

## 🚀 DevOps Relevance

`umask` is critical for:

* Secure file creation 🔐
* Application log protection 📜
* Compliance & hardening

Understanding umask prevents **silent security leaks** 💡.
