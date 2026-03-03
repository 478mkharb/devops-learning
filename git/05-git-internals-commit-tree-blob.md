# 🧠 Git Internals – Commit, Tree & Blob

This topic is **advanced but extremely impressive in DevOps interviews**. It explains **how Git actually stores data internally**, not just commands.

---

## 🌳 Big Picture (How Git Thinks)

Git is **not a file-based system** — it is a **content-addressable object database**.

Git stores everything as **objects**:

* **Blob** → file content
* **Tree** → directory structure
* **Commit** → snapshot + metadata

```
Commit → Tree → Blobs
```
<img width="987" height="544" alt="image" src="https://github.com/user-attachments/assets/4035175a-c136-4489-a26d-2f721cfc4e68" />

---

## 1️⃣ What is a Blob in Git?

### 📦 Blob = Binary Large Object

A **blob** stores the **actual content of a file**, but:

* ❌ It does NOT store filename
* ❌ It does NOT store permissions
* ✅ Only stores file data

### Example:

File: `app.py`

```python
print("Hello DevOps")
```

Git stores this content as a **blob object**:

```
blob a3f5c8...
```

📌 If two files have **same content**, Git stores **only one blob** (deduplication).

🎯 **Interview line:**

> "Blobs store content, not filenames."

---

## 2️⃣ What is a Tree in Git?

### 🌲 Tree = Directory Structure

A **tree object** represents:

* A directory
* Maps filenames → blobs or other trees

Tree contains:

* File names
* Blob hashes
* Permissions

### Example directory:

```
repo/
 ├── app.py
 └── config.yaml
```

Tree object:

```
tree 9d8e7f...
 ├── app.py → blob a3f5c8...
 └── config.yaml → blob b7c9d2...
```

📌 Trees allow Git to recreate folder hierarchy.

---

## 3️⃣ What is a Commit in Git?

### 🧾 Commit = Snapshot Pointer

A **commit object** contains:

* Reference to a **tree** (snapshot)
* Parent commit(s)
* Author & committer info
* Commit message
* Timestamp

### Example:

```bash
git commit -m "Add app config"
```

Internally:

```
commit e12a45...
 tree 9d8e7f...
 parent 6f7a21...
 author Mukesh
 message "Add app config"
```

📌 Commit does NOT store files directly — it points to a tree.

---

## 🔁 How They Work Together (Real Flow)

### Step 1: Create a file

```bash
echo "Hello" > app.txt
```

➡ Git creates a **blob** for file content

### Step 2: Stage the file

```bash
git add app.txt
```

➡ Git creates a **tree** representing directory state

### Step 3: Commit

```bash
git commit -m "Add app file"
```

➡ Git creates a **commit** pointing to the tree

```
Commit
  ↓
 Tree
  ↓
 Blob(s)
```

---

## 🧠 Visual Summary (Mental Model)

```
Commit
 ├── Metadata
 ├── Parent Commit
 └── Tree
      ├── file1 → Blob
      ├── file2 → Blob
      └── subdir → Tree
```

---

## 🔍 Why this matters in DevOps interviews

Understanding Git internals helps explain:

* Why Git is fast ⚡
* Why branches are lightweight 🌿
* Why rebasing rewrites history 🧬
* How deduplication works 💾

🎯 **Strong interview statement:**

> "Git stores snapshots using commits that point to trees, which map filenames to content blobs."

---

## 🧪 Optional: See objects yourself

```bash
git cat-file -t <hash>   # type
git cat-file -p <hash>   # content
```

---

## ✅ One-line Interview Cheat Sheet

* **Blob** → file content
* **Tree** → directory structure
* **Commit** → snapshot + metadata

---
