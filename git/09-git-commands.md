# Git Commands – Quick Explanations
---

## 🔄 Undo / Recovery Commands

### 1️⃣ `git reset`

Moves the branch pointer backward (rewrites history locally).

```bash
git reset HEAD~1
```

### 2️⃣ `git reset --soft`

Removes last commit but keeps changes staged.

```bash
git reset --soft HEAD~1
```

### 3️⃣ `git reset --mixed`

Removes commit and unstages changes (default behavior).

```bash
git reset HEAD~1
```

### 4️⃣ `git reset --hard`

Deletes commit and changes permanently.

```bash
git reset --hard HEAD~1
```

### 5️⃣ `git revert`

Safely undoes a commit by creating a new commit.

```bash
git revert HEAD
```

### 6️⃣ `git restore`

Restores files from staging or last commit.

```bash
git restore app.py
```

### 7️⃣ `git checkout -- <file>`

Discards local changes in a file (old style).

```bash
git checkout -- app.py
```

### 8️⃣ `git reflog`

Shows history of HEAD movements (used for recovery).

```bash
git reflog
```

---

## 🌿 Branching & Navigation

### 9️⃣ `git checkout`

Switches branches or checks out commits/files.

```bash
git checkout main
```

### 🔟 `git switch`

Safely switches branches (modern replacement).

```bash
git switch feature-login
```

### 1️⃣1️⃣ `git branch`

Creates, lists, or deletes branches.

```bash
git branch feature-api
```

### 1️⃣2️⃣ `git merge`

Combines another branch into the current branch.

```bash
git merge feature-api
```

### 1️⃣3️⃣ `git rebase`

Replays commits on top of another branch.

```bash
git rebase main
```

### 1️⃣4️⃣ `git cherry-pick`

Applies a specific commit from another branch.

```bash
git cherry-pick a1b2c3d
```

### 1️⃣5️⃣ `HEAD`

Pointer to the current commit/branch.

```bash
echo HEAD
```

### 1️⃣6️⃣ Detached HEAD

HEAD points to a commit instead of a branch.

```bash
git checkout a1b2c3d
```

---

## 📥 Remote & Sync Commands

### 1️⃣7️⃣ `git fetch`

Downloads changes without modifying local code.

```bash
git fetch origin
```

### 1️⃣8️⃣ `git pull`

Fetches and merges changes into current branch.

```bash
git pull origin main
```

### 1️⃣9️⃣ `git pull --rebase`

Fetches and rebases local commits on top.

```bash
git pull --rebase
```

### 2️⃣0️⃣ `git push`

Uploads local commits to remote repository.

```bash
git push origin main
```

### 2️⃣1️⃣ `git push --force`

Overwrites remote history (dangerous).

```bash
git push --force
```

### 2️⃣2️⃣ `git push --force-with-lease`

Force-pushes only if remote wasn’t updated by others.

```bash
git push --force-with-lease
```

### 2️⃣3️⃣ `git remote`

Manages remote repositories.

```bash
git remote -v
```

### 2️⃣4️⃣ `git remote add / set-url`

Adds or updates remote URL.

```bash
git remote set-url origin git@github.com:org/app.git
```

---

## 🧺 Staging Area

### 2️⃣5️⃣ `git add`

Stages changes for the next commit.

```bash
git add app.py
```

### 2️⃣6️⃣ `git add .`

Stages all changes in current directory.

```bash
git add .
```

### 2️⃣7️⃣ `git add -A`

Stages all changes including deletions.

```bash
git add -A
```

### 2️⃣8️⃣ `git reset <file>`

Unstages a file.

```bash
git reset app.py
```

### 2️⃣9️⃣ `git restore --staged`

Removes file from staging area.

```bash
git restore --staged app.py
```

---

## 🧳 Temporary Work

### 3️⃣0️⃣ `git stash`

Temporarily saves uncommitted changes.

```bash
git stash
```

### 3️⃣1️⃣ `git stash pop`

Restores and removes last stash.

```bash
git stash pop
```

### 3️⃣2️⃣ `git stash apply`

Restores stash without deleting it.

```bash
git stash apply
```

### 3️⃣3️⃣ `git stash drop`

Deletes a stash entry.

```bash
git stash drop
```

---

## 🏷️ Tags & Releases

### 3️⃣4️⃣ `git tag`

Creates a lightweight tag.

```bash
git tag v1.0
```

### 3️⃣5️⃣ `git tag -a`

Creates an annotated tag with metadata.

```bash
git tag -a v1.0 -m "release"
```

### 3️⃣6️⃣ `git push --tags`

Pushes all tags to remote.

```bash
git push --tags
```

### 3️⃣7️⃣ Lightweight vs Annotated Tag

Lightweight = pointer, Annotated = full object.

```bash
git tag -a v2.0
```

---

## 🪝 Hooks & Automation

### 3️⃣8️⃣ `pre-commit`

Runs before commit creation.

```bash
.git/hooks/pre-commit
```

### 3️⃣9️⃣ `post-commit`

Runs after commit is created.

```bash
.git/hooks/post-commit
```

### 4️⃣0️⃣ `pre-push`

Runs before pushing commits.

```bash
.git/hooks/pre-push
```

### 4️⃣1️⃣ `--no-verify`

Bypasses Git hooks.

```bash
git commit --no-verify
```

---

## 📂 Repo & Files

### 4️⃣2️⃣ `.gitignore`

Specifies files Git should ignore.

```bash
.env
```

### 4️⃣3️⃣ `.git/info/exclude`

Local ignore rules (not committed).

```bash
.git/info/exclude
```

### 4️⃣4️⃣ Tracked vs Untracked

Tracked = committed before, Untracked = new files.

```bash
git status
```

### 4️⃣5️⃣ Cached Files

Files already tracked by Git.

```bash
git rm --cached file.txt
```

---

## 🧠 Internals

### 4️⃣6️⃣ Git objects

Git stores data as blobs, trees, commits.

```bash
git cat-file -t <hash>
```

### 4️⃣7️⃣ Blob vs Tree vs Commit

Blob = content, Tree = structure, Commit = snapshot.

```bash
git cat-file -p <hash>
```

### 4️⃣8️⃣ `refs`

Pointers to commits (branches/tags).

```bash
.git/refs/heads/main
```

### 4️⃣9️⃣ `index`

Staging area stored as a file.

```bash
git ls-files --stage
```

### 5️⃣0️⃣ Reflog vs Log

Reflog = local history, Log = commit history.

```bash
git reflog
```

---

## 🎯 Final Interview Tip

> If you can explain **reset vs revert**, **fetch vs pull**, and **merge vs rebase** clearly — you will outperform most candidates.

---

