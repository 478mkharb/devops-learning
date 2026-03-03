## 🐧 What is ACL (Access Control List)?

**ACL (Access Control List)** is a Linux permission mechanism that provides **more fine‑grained access control** than traditional `rwx` permissions 🧠.

👉 In simple words: **ACL lets you give permissions to specific users or groups beyond owner, group, and others** 🔐.

---

## 🧠 Why ACL is Needed

Traditional Linux permissions support only:

* 👤 One owner
* 👥 One group
* 🌍 Others

❌ Problem:

* What if you want to give **read-only access to one extra user**?

✅ Solution:

* Use **ACL** to assign **custom permissions**.

---

## 🔍 When ACL is Used (Very Important)

ACL is useful when:

* Multiple users need different permissions
* Shared directories are used
* Application logs need controlled access
* Fine-grained security is required

---

## ⚙️ ACL Components

ACL can define permissions for:

* 👤 Specific users
* 👥 Specific groups
* 👑 File owner
* 👥 Owning group
* 🌍 Others
* 🎭 Mask (maximum effective permission)

---

## ▶️ Checking ACL Support

```bash
mount | grep acl
```

Most modern Linux filesystems (ext4, xfs) support ACL by default ✅

---

## 🔍 Viewing ACL of a File

```bash
getfacl file.txt
```

Example output:

```text
# file: file.txt
# owner: user1
# group: dev
user::rw-
user:user2:r--
group::r--
mask::r--
other::---
```

---

## 🧩 Understanding ACL Output

* `user::rw-` → Owner permissions
* `user:user2:r--` → Specific user access 👤
* `group::r--` → Group permissions
* `mask::r--` → Maximum allowed permissions 🎭
* `other::---` → Others have no access

---

## ➕ Setting ACL (Practical Example)

### 🔹 Scenario

* `file.txt` owned by `user1`
* Need to give **read-only access** to `user2`

### 🔹 Command

```bash
setfacl -m u:user2:r-- file.txt
```

Verify:

```bash
getfacl file.txt
```

➡️ `user2` can now read the file ✅

---

## 📁 ACL on Directories (VERY IMPORTANT)

### 🔹 Give write access to user

```bash
setfacl -m u:user2:rwx /data
```

---

## 🔁 Default ACL (Inheritance)

📌 Default ACL ensures **new files inherit permissions** inside a directory.

```bash
setfacl -d -m u:user2:rwX /data
```

➡️ Any new file inside `/data` will have ACL applied 🔄

---

## ❌ Removing ACL

```bash
setfacl -x u:user2 file.txt
```

Remove all ACLs:

```bash
setfacl -b file.txt
```

---

## 📊 ACL vs Traditional Permissions

| Feature          | chmod | ACL    |
| ---------------- | ----- | ------ |
| Multiple users   | ❌     | ✅      |
| Granular control | ❌     | ✅      |
| Inheritance      | ❌     | ✅      |
| Complexity       | Low   | Medium |

---

## 🧪 Real DevOps Example

🔹 Scenario:

* Jenkins runs as `jenkins` user
* Needs read access to app logs owned by `appuser`

🔹 Solution:

```bash
setfacl -m u:jenkins:r-- /var/log/app.log
```

➡️ No permission change for others
➡️ Secure and precise 🔐

---

## 🎯 Interview One-Liners

**Q: What is ACL?**
ACL allows assigning permissions to specific users or groups beyond standard Linux permissions.

**Q: Which commands manage ACL?**
`getfacl` and `setfacl`.

---

## 🚀 DevOps Relevance

ACL is critical for:

* Secure shared environments 🔐
* CI/CD tools access control 🤖
* Production log and data access

ACL prevents **over-permissioning** while maintaining flexibility 💡.
