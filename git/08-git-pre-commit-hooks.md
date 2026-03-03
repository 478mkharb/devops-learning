# 🪝 Git Hooks – Pre-Commit Hooks Explained

Git hooks are a **favorite interview topic** because they connect **Git internals, automation, CI/CD discipline, and DevOps mindset**.

---

## 🔹 What are Git hooks?

**Git hooks** are **scripts that Git automatically executes** at specific points in the Git lifecycle.

They allow you to:

* 🚫 Prevent bad commits
* 🧪 Run tests before code is committed
* 🔐 Enforce security & standards
* 🤖 Automate checks locally (shift-left)

📌 Hooks live inside the repository:

```
.git/hooks/
```

---

## 🔹 Types of Git hooks (High level)

Git hooks are broadly divided into two categories:

### 1️⃣ Client-side hooks (most common)

* `pre-commit`
* `prepare-commit-msg`
* `commit-msg`
* `pre-push`

### 2️⃣ Server-side hooks

* `pre-receive`
* `update`
* `post-receive`

📌 **DevOps interviews mostly focus on `pre-commit` hooks**.

---

## 🔹 What is a Git pre-commit hook?

A **pre-commit hook** runs:

⏱️ **Before** a commit is created

If the script:

* ✅ exits with `0` → commit continues
* ❌ exits with non-zero → commit is **blocked**

📌 Used to stop bad code *before it even enters Git history*.

---

## 🔹 Where is the pre-commit hook located?

```text
.git/hooks/pre-commit
```

Git provides a sample file:

```
pre-commit.sample
```

You must:

* Rename it to `pre-commit`
* Make it executable

---

## 🔹 How to create a custom Git pre-commit hook (Step-by-step)

### 🧩 Scenario

❌ We want to **block commits** if:

* A file contains the word `password`

This is a **real DevOps security use case** 🔐

---

### Step 1️⃣ Go to hooks directory

```bash
cd .git/hooks
```

---

### Step 2️⃣ Create the pre-commit hook

```bash
touch pre-commit
```

---

### Step 3️⃣ Add the script

```bash
#!/bin/bash

# Scan staged files for forbidden words
if git diff --cached | grep -i "password"; then
  echo "❌ Commit blocked: 'password' found in staged changes"
  exit 1
fi

exit 0
```

---

### Step 4️⃣ Make it executable

```bash
chmod +x pre-commit
```

---

## 🔹 How this hook works internally 🧠

```text
git commit
   ↓
pre-commit hook runs
   ↓
if exit 1 → commit fails
if exit 0 → commit succeeds
```

---

## 🔹 Test the pre-commit hook

### ❌ Bad commit attempt

```bash
echo "password=admin123" >> app.env
git add app.env
git commit -m "Add env config"
```

Output:

```
❌ Commit blocked: 'password' found in staged changes
```

---

### ✅ Good commit

```bash
echo "DB_USER=admin" >> app.env
git add app.env
git commit -m "Add db user"
```

✔ Commit succeeds 🎉

---

## 🔹 Common real-world uses of pre-commit hooks

* 🧪 Run unit tests
* 📏 Enforce formatting (linting)
* 🔐 Prevent secrets (AWS keys, passwords)
* 📝 Enforce commit message standards
* 🧹 Remove trailing spaces

---

## 🔹 Why Git hooks are important in DevOps

* 🛑 Catch issues **before CI/CD**
* ⚡ Faster feedback than pipelines
* 🔒 Improves security posture
* 📜 Enforces team standards locally

📌 Hooks = **Shift-left DevOps** mindset

---

## ⚠️ Important limitations of Git hooks

* ❌ Not shared automatically via Git
* ❌ Can be bypassed using `--no-verify`

```bash
git commit --no-verify
```

📌 To enforce hooks team-wide:

* Use tools like `pre-commit` framework
* Or enforce rules in CI pipelines

---

## 🎯 Interview Power Answer

> "Git pre-commit hooks are client-side scripts that run before a commit is created, allowing teams to enforce quality, security, and standards locally before code enters the repository."

---

## ✅ Quick Interview Cheat Sheet

* Hooks live in `.git/hooks/`
* `pre-commit` runs before commit creation
* Exit code controls success/failure
* Used for linting, tests, security
* Git hooks support DevOps automation

---
