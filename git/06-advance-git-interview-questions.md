# 🚀 Advanced Git & DevOps – Interview Answers

This section focuses on **advanced, DevOps-oriented Git concepts** that interviewers use to judge **real-world experience**, not just command knowledge.

---

## 1️⃣ What is `.gitignore` and why is it important?

`.gitignore` is a file that tells Git **which files or directories should NOT be tracked**.

### Why it is important:

* 🔐 Prevents committing secrets
* 🧹 Avoids clutter (logs, build files)
* ⚡ Keeps repo clean and lightweight

### Example `.gitignore`:

```gitignore
# Logs
*.log

# Environment files
.env

# Build output
dist/
node_modules/
```

📌 Files already tracked will not be ignored automatically.

---

## 2️⃣ How do you handle secrets in Git?

❌ **Never commit secrets** (passwords, tokens, keys).

### Best practices:

* Use `.gitignore` for secret files
* Use environment variables
* Use secret managers

  * AWS Secrets Manager
  * HashiCorp Vault
* Scan commits using hooks or CI tools

### Example:

```bash
export DB_PASSWORD=secret123
```

🎯 **Interview line:**

> "Secrets should live outside Git, injected at runtime."

---

## 3️⃣ What is Git tagging and why is it used?

A **Git tag** marks a **specific commit** as important (usually a release).

### Why used:

* 📦 Release management
* 🔁 Rollbacks
* 📜 Audit & traceability

### Example:

```bash
git tag v1.0.0
git push origin v1.0.0
```

📌 Tags are immutable snapshots.

---

## 4️⃣ What is semantic versioning in Git tags?

Semantic Versioning follows:

```
MAJOR.MINOR.PATCH
```

### Meaning:

* MAJOR → breaking changes 💥
* MINOR → backward-compatible features ✨
* PATCH → bug fixes 🐛

### Example:

```bash
v2.1.4
```

📌 Widely used in CI/CD pipelines.

---

## 5️⃣ How does Git help in CI/CD pipelines?

Git acts as the **single source of truth**.

### Git triggers:

* Commit → build
* PR → test
* Tag → release

### Example flow:

```
Git Push → CI Build → Test → Deploy
```

🎯 Git enables automation, traceability, and rollback.

---

## 6️⃣ What is a Git hook?

A **Git hook** is a script that runs **automatically** during Git events.

### Examples:

* `pre-commit` → before commit
* `pre-push` → before push
* `post-receive` → after push

📌 Hooks help enforce standards locally.

---

## 7️⃣ Difference between pre-commit and post-commit hooks

| Pre-Commit 🛑       | Post-Commit ✅          |
| ------------------- | ---------------------- |
| Runs before commit  | Runs after commit      |
| Can block commit    | Cannot block commit    |
| Used for validation | Used for notifications |

### Example:

```bash
# pre-commit → lint check
# post-commit → Slack notification
```

---

## 8️⃣ What is `git cherry-pick`?

`git cherry-pick` applies a **specific commit** from one branch to another.

### Example:

```bash
git cherry-pick a1b2c3d
```

📌 Used for hotfixes without merging entire branches.

---

## 9️⃣ What is `git stash` and its use cases?

`git stash` temporarily saves **uncommitted changes**.

### Example:

```bash
git stash
git checkout main
git stash pop
```

### Use cases:

* Quick context switching
* Clean working directory
* Emergency fixes

---

## 🔟 How do you manage large repositories in Git?

### Strategies:

* Use `.gitignore` aggressively
* Split repos (microservices)
* Use Git LFS for large files
* Shallow clones

### Example:

```bash
git clone --depth=1 repo.git
```

🎯 Keeps CI pipelines fast and efficient.

---

## 🧠 Interview Power Summary

> "Advanced Git practices help DevOps teams maintain secure, scalable, and automated delivery pipelines."

---

## ✅ One-Page Cheat Sheet

* `.gitignore` keeps repos clean
* Secrets live outside Git
* Tags = releases
* Git = CI/CD backbone
* Hooks = automation
* Cherry-pick = targeted fixes
* Stash = temporary shelf

---
