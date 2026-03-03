# Git Basics – Interview Answers

This document provides **detailed, interview‑ready explanations with examples** for the most common Git basics questions asked in **DevOps interviews**.

---

## 1. What is Git and why is it used in DevOps?

**Git** is a **distributed version control system (VCS)** used to track changes in source code, configuration files, and infrastructure code over time.

### Why Git is used in DevOps:

* Enables **collaboration** between developers, testers, and DevOps engineers
* Maintains **history and traceability** of every change
* Supports **CI/CD pipelines** by integrating with tools like Jenkins, GitHub Actions, GitLab CI
* Helps manage **Infrastructure as Code (IaC)** (Terraform, Ansible, Helm)
* Enables **rollback** to stable versions during failures

### Example (DevOps use case):

```bash
# Store application + Terraform code in Git
repo/
 ├── app/
 ├── Dockerfile
 ├── terraform/
 └── ansible/
```

Every infrastructure or app change is committed, reviewed, and deployed automatically.

---

## 2. What is the difference between Git and GitHub?

| Git                  | GitHub                                |
| -------------------- | ------------------------------------- |
| Version control tool | Hosting platform for Git repositories |
| Installed locally    | Cloud-based service                   |
| Manages code history | Adds collaboration & CI features      |
| CLI-based            | Web UI + APIs                         |

### Example:

```bash
# Git is used locally
git init
git commit -m "Initial commit"

# GitHub hosts the repo remotely
git push origin main
```

**In DevOps:** Git handles versioning, GitHub handles collaboration, PRs, and pipelines.

---

## 3. What is a repository in Git?

A **Git repository (repo)** is a directory that contains:

* Project files
* Complete change history
* Branches, tags, and metadata

There are two types:

* **Local repository** (on your machine)
* **Remote repository** (on GitHub/GitLab/Bitbucket)

### Example:

```bash
git init my-project
```

This creates a repository to track changes in `my-project`.

---

## 4. Difference between a local repo and a remote repo

| Local Repository      | Remote Repository      |
| --------------------- | ---------------------- |
| Exists on your system | Exists on server/cloud |
| Used for development  | Used for collaboration |
| Faster operations     | Shared among team      |

### Example flow:

```bash
# Local commit
git commit -m "Add Dockerfile"

# Push to remote
git push origin main
```

---

## 5. What is the Git workflow?

Git workflow defines **how changes move from developer to production**.

### Standard Git workflow:

1. Modify files
2. Stage changes (`git add`)
3. Commit changes (`git commit`)
4. Push to remote (`git push`)
5. Create PR & merge

### Diagram (logical):

```
Working Directory → Staging Area → Local Repo → Remote Repo
```

### Example:

```bash
git add app.py
git commit -m "Fix login bug"
git push origin main
```

---

## 6. What is the `.git` directory?

The `.git` directory is the **brain of Git**.
It stores:

* Commit history
* Branch references
* Configuration
* Hooks

### Important:

* Deleting `.git` removes version control
* Project files remain, history is lost

### Example:

```bash
ls -a
.git
```

---

## 7. What is a commit in Git?

A **commit** is a **snapshot of changes** with:

* Unique commit ID (SHA)
* Author
* Timestamp
* Commit message

### Example:

```bash
git commit -m "Add CI pipeline"
```

Each commit creates a restore point.

---

## 8. Difference between `git add` and `git commit`

| git add              | git commit                |
| -------------------- | ------------------------- |
| Stages changes       | Saves changes permanently |
| Prepares snapshot    | Creates snapshot          |
| Can be undone easily | Becomes part of history   |

### Example:

```bash
git add Dockerfile
git commit -m "Add Docker support"
```

---

## 9. What is `git status` used for?

`git status` shows the **current state of the working tree**:

* Modified files
* Staged files
* Untracked files
* Current branch

### Example:

```bash
git status
```

Output helps DevOps engineers avoid pushing unintended files.

---

## 10. What is `git log` and how do you customize it?

`git log` displays **commit history**.

### Basic usage:

```bash
git log
```

### Useful customizations:

```bash
# One-line summary
git log --oneline

# Show last 3 commits
git log -3

# Graph view
git log --oneline --graph --all

# Custom format
git log --pretty=format:"%h %an %s"
```

**DevOps Tip:** `git log --oneline --graph` is commonly used during incident analysis.

---

## Interview Tip (How to Answer Confidently)

> "Git is the backbone of DevOps because it provides versioning, collaboration, auditability, and seamless CI/CD integration."

---
