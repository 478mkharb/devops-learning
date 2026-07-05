# Git Squash

> **Interview Tip:** Git Squash is used to **combine multiple commits into a single commit**. It helps keep the Git history clean and readable before merging a feature branch into the main branch.

---

# What is Git Squash?

Git Squash is the process of **merging multiple commits into one single commit**.

Instead of having many small commits like:

```text
Added Login Page

Fixed Login Bug

Updated CSS

Fixed Typo

Added Validation
```

Git Squash combines them into:

```text
Added Login Feature
```

---

# Before Squash

```text
main

A──B──C

         \
feature    D──E──F──G
```

Commits:

```text
D → Created Login Page

E → Fixed CSS

F → Added Validation

G → Fixed Typo
```

---

# After Squash

```text
main

A──B──C

         \
feature    H
```

Where

```text
H

Added Complete Login Feature
```

All four commits become **one commit**.

---

# Why Do We Use Git Squash?

- Keep Git history clean
- Reduce unnecessary commits
- Make Pull Requests easier to review
- Improve readability
- Simplify rollback
- Maintain meaningful commit history

---

# How to Squash Commits?

Suppose the last **4 commits** need to be squashed.

### Step 1

```bash
git rebase -i HEAD~4
```

---

### Step 2

Git opens an editor.

Before

```text
pick a1b2c3 Created Login

pick d4e5f6 Fixed CSS

pick g7h8i9 Added Validation

pick j1k2l3 Fixed Typo
```

---

### Step 3

Change

```text
pick
```

to

```text
squash
```

or

```text
s
```

Result

```text
pick a1b2c3 Created Login

s d4e5f6 Fixed CSS

s g7h8i9 Added Validation

s j1k2l3 Fixed Typo
```

---

### Step 4

Git asks for a commit message.

Example

```text
Added complete Login Feature
```

---

### Final History

```text
main

A──B──C

         \
feature    H
```

---

# Git Squash Merge

GitHub, GitLab, and Bitbucket also provide a **Squash and Merge** option.

Instead of

```text
main

A──B──C────────M

         \    /

          D──E──F──G
```

You get

```text
main

A──B──C──H
```

Only **one commit** is added to `main`.

---

# Merge vs Rebase vs Squash

| Feature | Git Merge | Git Rebase | Git Squash |
|---------|-----------|------------|------------|
| Purpose | Combine branches | Rewrite commit history | Combine multiple commits into one |
| Merge Commit | ✅ Yes | ❌ No | ❌ No |
| History | Preserved | Rewritten | Simplified |
| Commit Hash Changes | No | Yes | Yes |
| Creates Linear History | No | Yes | Yes |
| Best Use | Team collaboration | Clean feature history | Clean Pull Requests |

---

# Example

Without Squash

```text
main

A──B──C──D──E──F──G
```

Git Log

```text
Fix typo

Updated CSS

Fixed API

Added Validation

Login Feature
```

---

After Squash

```text
main

A──B──C──H
```

Git Log

```text
Added Login Feature
```

---

# Advantages

- Clean commit history
- Easier code reviews
- Better Git logs
- One meaningful commit per feature
- Easier rollback

---

# Disadvantages

- Individual commit history is lost
- Harder to trace intermediate development steps
- Changes commit hashes

---

# Common Interview Questions

| Question | Answer |
|----------|--------|
| What is Git Squash? | Git Squash combines multiple commits into a single commit to keep the Git history clean. |
| Why is Git Squash used? | To reduce unnecessary commits and improve repository readability. |
| Does Git Squash change commit hashes? | Yes. A new commit is created, so the commit hash changes. |
| Does Git Squash create a merge commit? | No. It creates a single new commit representing all squashed commits. |
| Which command is used for squashing commits? | `git rebase -i HEAD~<number_of_commits>` |
| Can GitHub perform Squash automatically? | Yes, using the **Squash and Merge** option in a Pull Request. |
| When should you use Git Squash? | Before merging a feature branch to keep commit history concise and meaningful. |

---

# Merge vs Rebase vs Squash (Interview Summary)

| Git Merge | Git Rebase | Git Squash |
|------------|------------|------------|
| Combines branches | Replays commits on top of another branch | Combines multiple commits into one |
| Preserves history | Rewrites history | Simplifies history |
| Creates merge commit | No merge commit | No merge commit |
| Best for shared branches | Best for local feature branches | Best before merging Pull Requests |

---

# One-Line Interview Answer

> **Git Squash is the process of combining multiple commits into a single meaningful commit, resulting in a cleaner and more readable Git history before merging a feature branch into the main branch.**
