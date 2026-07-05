# Git Merge vs Git Rebase

> **Interview Tip:** Both **Merge** and **Rebase** are used to integrate changes from one branch into another. The main difference is **Merge preserves branch history**, while **Rebase rewrites commit history to create a linear history**.

---

# Git Merge vs Git Rebase

| Feature | Git Merge | Git Rebase |
|---------|-----------|------------|
| **Definition** | Combines changes from one branch into another by creating a **new merge commit**. | Moves or reapplies commits from one branch onto another, creating a **linear history**. |
| **History** | Preserves complete branch history. | Rewrites commit history. |
| **Merge Commit** | ✅ Creates a merge commit (unless fast-forward). | ❌ No merge commit. |
| **Commit IDs** | Original commit hashes remain unchanged. | New commit hashes are created. |
| **History Structure** | Non-linear (branching history). | Linear (straight-line history). |
| **Readability** | Shows how branches evolved. | Cleaner and easier to read. |
| **Risk** | Safe because history is preserved. | Can be risky if used on shared/public branches. |
| **Conflict Resolution** | Resolve conflicts once during merge. | May need to resolve conflicts for each rebased commit. |
| **Best For** | Shared branches, team collaboration, production branches. | Cleaning local commit history before creating a Pull Request. |
| **Recommended On** | Shared branches (`main`, `develop`) | Local feature branches only. |

---

# Git Merge

## What is Git Merge?

Git Merge combines two branches and creates a **merge commit**, preserving the complete history of both branches.

### Diagram

```text
Before Merge

main
A──B──C────────────

         \
feature    D──E


After Merge

main
A──B──C────────────M
         \        /
feature    D────E
```

`M` = Merge Commit

---

### Command

```bash
git checkout main
git merge feature
```

---

### Advantages

- Preserves complete history
- Safe for shared branches
- Easy to understand
- No history rewriting

---

### Disadvantages

- Creates additional merge commits
- Repository history can become cluttered
- Harder to follow in large projects

---

# Git Rebase

## What is Git Rebase?

Git Rebase moves the commits of one branch on top of another branch, creating a **linear commit history**.

### Diagram

```text
Before Rebase

main
A──B──C────────────

         \
feature    D──E


After Rebase

main
A──B──C──D──E
```

Notice:

- No merge commit
- Commits **D** and **E** are recreated after **C**

---

### Command

```bash
git checkout feature
git rebase main
```

Then

```bash
git checkout main
git merge feature
```

This merge becomes a **Fast-Forward Merge**.

---

### Advantages

- Clean, linear history
- Easier to read logs
- No unnecessary merge commits
- Preferred before creating Pull Requests

---

### Disadvantages

- Rewrites commit history
- Changes commit hashes
- Dangerous on shared/public branches
- May require resolving conflicts multiple times

---

# Visual Comparison

## Git Merge

```text
          feature
          D──E
         /
A──B──C────────M
               │
             main
```

---

## Git Rebase

```text
A──B──C──D──E
             │
           main
```

---

# When to Use Git Merge

✅ Shared branches

✅ Production branches

✅ Team collaboration

✅ Preserve complete history

Examples:

- main
- develop
- release

---

# When to Use Git Rebase

✅ Local feature branches

✅ Before creating Pull Request

✅ Cleaning commit history

✅ Updating feature branch with latest main

Example:

```bash
git checkout feature

git fetch origin

git rebase origin/main
```

---

# Merge vs Rebase Example

Suppose

```text
main

A──B──C

feature

A──B──D──E
```

## Merge

```bash
git checkout main

git merge feature
```

Result

```text
A──B──C────M
     \    /
      D──E
```

---

## Rebase

```bash
git checkout feature

git rebase main
```

Result

```text
A──B──C──D'──E'
```

Notice:

- D and E become D' and E'
- Commit hashes change

---

# Common Interview Questions

| Question | Answer |
|----------|--------|
| What is Git Merge? | Git Merge combines two branches and preserves branch history by creating a merge commit. |
| What is Git Rebase? | Git Rebase reapplies commits from one branch onto another, creating a linear commit history. |
| Does Git Merge create a merge commit? | Yes (unless it's a fast-forward merge). |
| Does Git Rebase create a merge commit? | No. |
| Does Git Rebase change commit hashes? | Yes. |
| Does Git Merge change commit hashes? | No. |
| Which command preserves history? | Git Merge. |
| Which command creates a linear history? | Git Rebase. |
| Is Git Rebase safe on shared branches? | No. It should generally be avoided on public/shared branches because it rewrites history. |
| Which is preferred before creating a Pull Request? | Git Rebase (to clean local history), though many teams also use merge depending on their workflow. |

---

# Merge vs Rebase Summary

| Use Merge When... | Use Rebase When... |
|-------------------|--------------------|
| Working with shared branches | Working on your local feature branch |
| You want to preserve history | You want a clean linear history |
| Team collaboration | Preparing a Pull Request |
| Production branch | Updating feature branch with latest `main` |

---

# Interview One-Liner

> **Git Merge combines branches while preserving branch history by creating a merge commit, whereas Git Rebase rewrites commit history by replaying commits on top of another branch, resulting in a clean, linear history without a merge commit.**
