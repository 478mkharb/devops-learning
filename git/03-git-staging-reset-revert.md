# 🧰 Git Staging, Reset & Revert – Interview Answers

This section explains **one of the most confusing but most-asked Git topics** in DevOps interviews: **staging, undoing changes, and recovery** — with **clear concepts, commands, and real scenarios**.

---

## 1️⃣ What is the staging area?

The **staging area (index)** is an **intermediate layer** between your working directory and the Git repository.

📌 It allows you to **choose exactly what goes into the next commit**.

### Git areas flow:

```
Working Directory → Staging Area → Repository
```

### Example:

```bash
git add app.py
```

Only `app.py` is staged — other modified files are ignored.

🧠 **Interview insight:** Staging enables **atomic commits**, which is critical for CI/CD traceability.

---

## 2️⃣ Difference between `git reset` and `git revert`

| git reset 🔄                 | git revert ↩️        |
| ---------------------------- | -------------------- |
| Moves branch pointer         | Creates a new commit |
| Rewrites history             | Preserves history    |
| Dangerous on shared branches | Safe for production  |
| Used locally                 | Used in production   |

### Example:

```bash
# Rewrite history (local)
git reset --hard HEAD~1

# Safe undo (prod)
git revert HEAD
```

📌 **Golden rule:** Never use `reset` on shared branches.

---

## 3️⃣ Explain `git reset --soft`, `--mixed`, and `--hard`

### `--soft` 🟢

* Undo commit
* Changes stay **staged**

```bash
git reset --soft HEAD~1
```

### `--mixed` 🟡 (default)

* Undo commit
* Changes stay **unstaged**

```bash
git reset HEAD~1
```

### `--hard` 🔴

* Undo commit
* Deletes changes permanently ⚠️

```bash
git reset --hard HEAD~1
```

🧠 **Memory trick:** Soft = commit only, Mixed = commit + stage, Hard = everything.

---

## 4️⃣ When would you use `git revert` in production?

You use `git revert` when:

* Code is already **pushed**
* CI/CD pipeline is active
* Multiple people depend on the branch

### Example (production fix):

```bash
git revert a1b2c3d
```

✅ Creates a new commit that safely undoes changes.

🎯 **Interview line:**

> "Revert is audit-safe and production-safe."

---

## 5️⃣ How do you undo the last commit?

### If NOT pushed:

```bash
git reset --soft HEAD~1
```

### If already pushed:

```bash
git revert HEAD
```

📌 Choose based on **branch visibility**.

---

## 6️⃣ How do you unstage a file?

### Command:

```bash
git restore --staged app.py
```

### Old way:

```bash
git reset app.py
```

📌 File remains modified, just removed from staging.

---

## 7️⃣ What is `git checkout` used for?

`git checkout` is a **multi-purpose command** used to:

* Switch branches
* Restore files
* Checkout commits

### Examples:

```bash
git checkout main
git checkout -- config.yaml
```

⚠️ Because of confusion, Git introduced `switch` and `restore`.

---

## 8️⃣ Difference between `git checkout` and `git switch`

| git checkout            | git switch        |
| ----------------------- | ----------------- |
| Old & overloaded        | Modern & focused  |
| Used for many things    | Only for branches |
| Confusing for beginners | Safer & clearer   |

### Example:

```bash
git switch feature-login
```

---

## 9️⃣ What is `git restore`?

`git restore` is used to **discard or unstage changes**.

### Examples:

```bash
# Discard changes
git restore app.py

# Unstage changes
git restore --staged app.py
```

🧠 Separates responsibilities cleanly from `checkout`.

---

## 🔟 How do you recover a deleted commit?

Even deleted commits can be recovered using **reflog**.

### Steps:

```bash
git reflog
```

Find commit hash, then:

```bash
git checkout -b recovery-branch <commit-id>
```

✅ Commit restored successfully 🎉

📌 **Reflog is local and temporary** — act fast.

---

## 🎯 Interview Power Summary

> "Reset rewrites history, revert preserves it, and reflog saves you when things go wrong."

---
