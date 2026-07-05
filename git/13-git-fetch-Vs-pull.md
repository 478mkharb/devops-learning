# Git Fetch vs Git Pull

> **Interview Tip:** Both `git fetch` and `git pull` are used to get the latest changes from a remote repository. The key difference is that **`git fetch` only downloads changes**, whereas **`git pull` downloads and automatically integrates them into your current branch**.

---

# Git Fetch vs Git Pull

| Feature | Git Fetch | Git Pull |
|---------|-----------|-----------|
| **Definition** | Downloads the latest changes from the remote repository without modifying the local working branch. | Downloads the latest changes and automatically merges (or rebases) them into the current local branch. |
| **Updates Local Branch** | ❌ No | ✅ Yes |
| **Updates Remote Tracking Branch** | ✅ Yes | ✅ Yes |
| **Automatic Merge** | ❌ No | ✅ Yes (or rebase if configured) |
| **Working Directory Changes** | No | Yes |
| **Risk of Merge Conflicts** | None during fetch | Possible during merge/rebase |
| **Safe to Use Anytime** | ✅ Yes | ⚠️ Only when ready to integrate changes |
| **Common Use Case** | Review remote changes before merging | Synchronize local branch with remote branch |
| **Command** | `git fetch origin` | `git pull origin main` |

---

# Git Fetch

## What is Git Fetch?

Git Fetch **downloads the latest commits, branches, and tags from the remote repository** but **does not merge them** into your current branch.

### Diagram

```text
Before Fetch

Remote Repository

main

A──B──C──D


Local Repository

main

A──B──C
```

Run

```bash
git fetch origin
```

After Fetch

```text
Remote Tracking Branch

origin/main

A──B──C──D


Local Branch

main

A──B──C
```

**Notice:** Local branch is **unchanged**.

---

## Advantages

- Safe operation
- No automatic merge
- Review changes before integrating
- No risk of unexpected conflicts

---

## Common Commands

```bash
git fetch
```

Fetch all remotes

```bash
git fetch --all
```

View fetched commits

```bash
git log HEAD..origin/main
```

See differences

```bash
git diff main origin/main
```

---

# Git Pull

## What is Git Pull?

Git Pull **downloads the latest changes and immediately merges (or rebases) them into your current branch**.

It is equivalent to:

```bash
git fetch

+

git merge
```

(or `git rebase`, if configured)

---

### Diagram

Before Pull

```text
Remote

A──B──C──D


Local

A──B──C
```

Run

```bash
git pull origin main
```

After Pull

```text
Local

A──B──C──D
```

Local branch is immediately updated.

---

## Advantages

- Quick synchronization
- One command
- Convenient for daily development

---

## Disadvantages

- May create merge conflicts immediately
- Less control over incoming changes

---

# Internal Working

## Git Fetch

```text
git fetch

↓

Downloads commits

↓

Updates origin/main

↓

Stops
```

---

## Git Pull

```text
git pull

↓

git fetch

↓

git merge

(or git rebase)

↓

Updates local branch
```

---

# Example

Current Repository

```text
Remote

A──B──C──D


Local

A──B──C
```

---

### Using Fetch

```bash
git fetch origin
```

Result

```text
origin/main

A──B──C──D

main

A──B──C
```

No merge happens.

---

### Using Pull

```bash
git pull origin main
```

Result

```text
main

A──B──C──D
```

Changes are merged automatically.

---

# When to Use Git Fetch

- Before starting work
- Before creating a Pull Request
- To inspect incoming changes
- In production repositories
- When you don't want automatic merges

---

# When to Use Git Pull

- Daily synchronization
- Small teams
- When you trust remote changes
- Before continuing development
- When automatic merging is acceptable

---

# Common Interview Questions

| Question | Answer |
|----------|--------|
| What is Git Fetch? | Git Fetch downloads the latest changes from the remote repository without modifying the current local branch. |
| What is Git Pull? | Git Pull downloads the latest changes and automatically merges (or rebases) them into the current local branch. |
| Does Git Fetch modify the working directory? | No. |
| Does Git Pull modify the working directory? | Yes. |
| Can Git Fetch cause merge conflicts? | No, because it doesn't merge changes. |
| Can Git Pull cause merge conflicts? | Yes, during the merge or rebase step. |
| Which command is safer? | Git Fetch, because it only downloads changes. |
| What does Git Pull internally execute? | `git fetch` followed by `git merge` (or `git rebase` if configured). |
| Which command is recommended before reviewing incoming changes? | Git Fetch. |

---

# Quick Comparison

| Git Fetch | Git Pull |
|------------|-----------|
| Downloads changes only | Downloads and integrates changes |
| No merge | Automatic merge/rebase |
| Safe | Can cause merge conflicts |
| Local branch unchanged | Local branch updated |
| Best for reviewing changes | Best for quick synchronization |

---

# One-Line Interview Answer

> **Git Fetch downloads the latest changes from the remote repository without modifying the current branch, whereas Git Pull downloads the changes and immediately integrates them into the current branch by performing a merge (or rebase).**
