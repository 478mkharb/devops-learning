# Git Senior L2 Interview Preparation

A practical **Senior L2 Git interview guide** focused on Git internals, branching, merging, rebasing, recovery, remotes, troubleshooting, security, and CI/CD.

> **Interview principle:** Do not answer only with commands. Explain **what Git state you are inspecting, why you chose a command, what risk it has, how you verify the result, and how you prevent recurrence.**

---

# 1. Git Fundamentals

## Q1. What is Git?

### Short Interview Answer

Git is a **distributed version control system (DVCS)** that records changes to files as a history of commits. Each clone normally contains the repository's history and can create commits locally without a central server.

### Detailed Explanation

Git tracks the evolution of a project through **commits**. A repository contains Git's object database and references such as branches and tags.

Because Git is distributed:

- Developers can commit offline.
- Branches can be created cheaply.
- History can be inspected locally.
- Local history can be rewritten when appropriate.
- Collaboration is performed through remotes such as GitHub, GitLab, or Bitbucket.

### Important Distinction

Git itself is the version-control system. GitHub/GitLab/Bitbucket are hosting and collaboration platforms built around Git.

### Interview Point

> Git is distributed, so the local clone is a complete working repository rather than merely a client connected to a central history server.

---

## Q2. What is the difference between Git and GitHub?

| Git | GitHub |
|---|---|
| Version-control system | Hosting/collaboration platform |
| Runs locally | Cloud/service platform |
| Tracks commits and history | Hosts repositories and adds collaboration features |
| Works without GitHub | Uses Git underneath |
| Provides commands such as `commit`, `branch`, `merge` | Provides PRs, reviews, Actions, permissions, branch rules, etc. |

### Interview Answer

> Git is the version-control technology. GitHub is a platform that hosts Git repositories and provides collaboration, code review, CI/CD, permissions, and other services.

Git can be used without GitHub.

---

# 2. Git Architecture and State

## Q3. Explain the working tree, staging area, and repository.

Git's local workflow can be visualized as:

```text
             git add
Working Tree ─────────→ Staging Area / Index
                              │
                              │ git commit
                              ↓
                       Local Repository
```

### Working Tree

The files currently checked out on disk. You modify these files while developing.

### Staging Area / Index

The exact content selected for the **next commit**.

This is why:

```bash
git add app.py
```

does not mean "commit `app.py`". It means "put the current version of `app.py` into the proposed next snapshot."

### Repository

The `.git` database contains objects and references representing commits, trees, blobs, tags, and branch state.

### Useful Commands

```bash
git status
git add <file>
git diff
git diff --staged
git commit -m "message"
```

### Interview Trap

`git diff` and `git diff --staged` do not show the same thing.

---

## Q4. What happens internally when you run `git commit`?

At a high level:

```text
Working Tree
     │
     │ git add
     ↓
Index / Staging Area
     │
     │ git commit
     ↓
Tree + Commit objects
     │
     ↓
Current branch reference moves
```

Git uses the index to determine the tree that the new commit should represent. It creates/reuses the necessary content objects, creates a commit object containing metadata and a reference to the tree, and records the parent commit(s).

A commit normally contains:

- Parent commit(s)
- Tree reference
- Author
- Committer
- Commit message
- Timestamps/metadata

The branch itself does not contain the file contents; it is a reference to a commit.

### Interview Point

> A commit is a snapshot represented by Git objects and metadata. Creating a commit also advances the current branch reference to the new commit.

---

# 3. Git Objects

## Q5. What are Git's main object types?

Git's core object types are:

| Object | Purpose |
|---|---|
| **Blob** | File content |
| **Tree** | Directory structure and references to blobs/trees |
| **Commit** | Points to a tree and parent commit(s), plus metadata |
| **Tag** | Annotated tag object pointing to another Git object |

### Blob

A blob stores file content. It does not itself store the filename.

### Tree

A tree records names, modes, and references to blobs or other trees.

### Commit

A commit points to a root tree and one or more parents.

### Tag

An **annotated tag** is a Git object containing information such as tagger, message, target object, and optionally a signature.

A **lightweight tag** is simply a reference and is not a tag object.

### Useful Inspection

```bash
git cat-file -t <object-id>
git cat-file -p <object-id>
```

---

## Q6. What is a Git hash/object ID?

Git identifies objects using **content-derived object IDs**.

Repositories may use SHA-1 or SHA-256 depending on repository format.

For an object, its object ID is derived from its type and content. If the relevant content changes, the resulting object ID changes.

This gives Git:

- Content addressing
- Object identity
- Change detection
- Efficient object reuse/deduplication

### Commands

```bash
git rev-parse HEAD
git cat-file -t <object-id>
git cat-file -p <object-id>
```

### Interview Trap

Do not say that every Git "hash" is simply a hash of the file. A commit ID identifies a **commit object**, whose content includes references such as its tree and parents.

---

# 4. Branching

## Q7. What is a Git branch?

A branch is essentially a **movable reference to a commit**.

Example:

```text
A---B---C   main
     \
      D---E feature
```

`main` points to `C`; `feature` points to `E`.

Creating a branch does not copy the complete repository:

```bash
git switch -c feature/login
```

It creates a new reference pointing at the current commit and switches `HEAD` to that branch.

### Interview Point

> A branch is a pointer/reference, not a separate copy of the repository.

---

## Q8. What is the difference between `git checkout` and `git switch`?

`git checkout` is an older multi-purpose command. It can switch branches, create branches, detach `HEAD`, and restore paths.

`git switch` was introduced specifically for branch switching and is clearer for modern workflows.

```bash
git switch main
git switch -c feature/login
```

For restoring files:

```bash
git restore <file>
```

### Practical Rule

Prefer:

```text
git switch → branch operations
git restore → file restoration
```

`checkout` is still valid and widely encountered in existing repositories.

---

# 5. Merge

## Q9. What is Git merge?

`git merge` integrates the history of another branch into the current branch.

Example:

```text
A---B---C---D   main
     \
      E---F     feature
```

If you run:

```bash
git switch main
git merge feature
```

Git may create:

```text
A---B---C---D---M   main
     \         /
      E---F----
```

`M` is a merge commit when a true merge is required.

### Important

Merge **does not normally rewrite the existing commits** on the branches being merged.

---

## Q10. What is a fast-forward merge?

A fast-forward is possible when the current branch is an ancestor of the branch being merged.

Before:

```text
A---B---C   main
         \
          D---E   feature
```

If `main` has not moved:

```bash
git switch main
git merge feature
```

Git can simply move `main`:

```text
A---B---C---D---E   main
```

No merge commit is created.

### Interview Point

> Fast-forward is not really a new merge commit; Git simply advances the target reference.

---

## Q11. What is `git merge --no-ff`?

`--no-ff` tells Git to create a merge commit even when a fast-forward would otherwise be possible.

```bash
git merge --no-ff feature
```

Conceptually:

```text
A---B---C---------M main
         \       /
          D---E
```

This can preserve an explicit branch integration point.

### Interview Trap

`--no-ff` does **not** preserve the branch's original name forever. It preserves a merge commit representing the integration event.

---

# 6. Merge Conflicts

## Q12. What causes a merge conflict and how do you resolve it?

A conflict occurs when Git cannot automatically determine a valid result for overlapping changes.

Typical case:

```text
main:    changes line 20 to A
feature: changes line 20 to B
```

Git cannot safely choose A or B.

Check:

```bash
git status
```

You may see conflict markers:

```text
<<<<<<< HEAD
current branch version
=======
incoming branch version
>>>>>>> feature
```

Resolve the file manually, remove the markers, then stage it:

```bash
git add <resolved-file>
```

For a merge:

```bash
git commit
```

To abandon an in-progress merge:

```bash
git merge --abort
```

### Important

Do not blindly use:

```bash
git add .
```

during a conflict. Review exactly what you are staging.

---

# 7. Rebase

## Q13. What is Git rebase?

Rebase changes the base of a sequence of commits by replaying those commits on another base.

Before:

```text
A---B---C   main
     \
      D---E feature
```

Run:

```bash
git switch feature
git rebase main
```

Conceptually:

```text
A---B---C---D'---E'   feature
```

`D'` and `E'` are newly created commits with different IDs.

### Why?

A commit includes its parent relationship. Changing the parent changes the commit's content and therefore its object ID.

### Interview Point

> Rebase rewrites commit history. The changes may be logically identical, but the rewritten commits are new commits.

---

## Q14. Merge vs rebase?

| Merge | Rebase |
|---|---|
| Integrates histories | Replays commits onto another base |
| Can create a merge commit | Produces rewritten commits |
| Preserves existing commit IDs | Rewritten commits get new IDs |
| Preserves topology | Can produce a linear history |
| Usually safer for shared public history | Best suited to private/local work unless team policy says otherwise |

### Practical Rule

> Rebase local/private work when it improves history; avoid rebasing commits that others are already building on unless the team explicitly coordinates it.

---

## Q15. What does `git rebase -i` do?

Interactive rebase lets you modify a sequence of commits.

```bash
git rebase -i HEAD~5
```

Common actions:

| Action | Meaning |
|---|---|
| `pick` | Keep commit |
| `reword` | Keep changes, edit message |
| `edit` | Stop and amend commit |
| `squash` | Combine with previous commit and edit combined message |
| `fixup` | Combine with previous commit, discard this commit's message |
| `drop` | Remove commit from the rebased sequence |

It is useful for cleaning up **local history before sharing it**.

---

# 8. Rebase Conflicts

## Q16. You are rebasing and get conflicts. What do you do?

First inspect:

```bash
git status
```

Resolve each conflict, then:

```bash
git add <resolved-file>
git rebase --continue
```

Repeat until complete.

If you decide to abandon the rebase:

```bash
git rebase --abort
```

### If you need to skip a problematic commit

```bash
git rebase --skip
```

Use this only when you understand why that commit can be skipped.

### Interview Point

During rebase, `git status` is your primary state-inspection command. Do not blindly run `git commit`; `git rebase --continue` controls the rebase sequence.

---

# 9. Reset

## Q17. Explain `git reset --soft`, `--mixed`, and `--hard`.

`git reset` moves a branch/`HEAD` reference and can also update the index and working tree depending on the mode.

| Mode | HEAD/branch | Index | Working tree |
|---|---|---|---|
| `--soft` | Moves | Unchanged | Unchanged |
| `--mixed` | Moves | Reset | Unchanged |
| `--hard` | Moves | Reset | Reset |

### Soft

```bash
git reset --soft HEAD~1
```

The previous commit is removed from the branch history, but its changes remain staged.

Useful when you want to redo a commit.

### Mixed

```bash
git reset HEAD~1
```

`--mixed` is the default. It moves the branch and resets the index, while keeping working-tree changes.

### Hard

```bash
git reset --hard HEAD~1
```

Moves the branch and resets the index and tracked working tree to match the target commit.

> **Warning:** `--hard` can discard uncommitted tracked changes.

### Interview Trap

Reset does not inherently "delete a commit object immediately." It moves references. The old commit may remain recoverable through reflog until unreachable objects are eventually pruned.

---

# 10. Revert

## Q18. What is the difference between reset and revert?

| `reset` | `revert` |
|---|---|
| Moves a branch/reference | Creates a new commit |
| Can rewrite local branch history | Preserves existing history |
| Useful for local history cleanup | Usually preferred for shared branches |
| Can change index/worktree depending on mode | Applies inverse changes |

Example:

```text
A---B---C---D
```

Run:

```bash
git revert C
```

Result:

```text
A---B---C---D---R
```

`R` is a new commit that reverses the changes introduced by `C`.

### Senior L2 Answer

> On a shared branch, I normally prefer `revert` because it preserves the published history and records the rollback explicitly. I use `reset` primarily when I intentionally need to move a local/private branch.

---

# 11. Recovery and Reflog

## Q19. You accidentally ran `git reset --hard`. Can the lost commit be recovered?

Often, yes, provided the commit/object has not become permanently unavailable through repository cleanup.

Start with:

```bash
git reflog
```

Example:

```text
HEAD@{0} commit ...
HEAD@{1} commit ...
HEAD@{2} commit ...
```

Find the previous commit and inspect it:

```bash
git show <commit>
```

Create a recovery branch before doing anything destructive:

```bash
git switch -c recovery <commit>
```

### Senior L2 Approach

1. Stop destructive operations.
2. Inspect reflog.
3. Identify the desired commit.
4. Create a recovery reference.
5. Verify the recovered history.
6. Only then restore the intended branch.

---

## Q20. What is reflog?

Reflog records **local movements of references**, such as `HEAD` and local branches.

```bash
git reflog
git reflog show main
```

It is extremely useful after:

- `reset`
- `rebase`
- branch movement
- accidental checkout/reset
- recovering a previously referenced commit

### Important Limitation

Reflog is primarily **local**. A remote repository does not automatically contain your local reflog.

So:

> Reflog is a recovery aid, not a substitute for remote backups.

---

# 12. Remote Repositories

## Q21. What is the difference between `git fetch` and `git pull`?

`git fetch` downloads remote updates and updates remote-tracking references without integrating those changes into your current branch.

```bash
git fetch origin
```

`git pull` generally performs:

```text
git fetch
   +
integration (merge or rebase)
```

The exact integration behavior depends on configuration and options.

### Safer investigation workflow

```bash
git fetch origin
git log HEAD..origin/main --oneline
```

Then decide whether to merge or rebase.

---

## Q22. Why is `git fetch` useful for troubleshooting?

Because it updates information about the remote without automatically modifying your current branch.

For example:

```bash
git fetch origin
git log HEAD..origin/main --oneline
git diff HEAD..origin/main
```

Now you can understand what changed remotely before integrating it.

### Interview Point

> `fetch` separates **information gathering** from **history integration**.

---

## Q23. What is `origin`?

`origin` is simply the conventional default name Git assigns to the remote from which a repository was cloned.

Check:

```bash
git remote -v
```

You can have multiple remotes:

```bash
git remote add upstream <url>
```

For example:

```text
origin   → your fork
upstream → original project
```

The name itself has no special protocol meaning.

---

# 13. Push and Force Push

## Q24. What is the difference between `git push` and `git push --force`?

Normal push generally requires the remote branch to be updated in a way Git considers a fast-forward (unless special ref rules apply).

```bash
git push
```

A force push permits rewriting the remote branch reference:

```bash
git push --force
```

This can remove commits from the remote branch's visible history.

### Safer Alternative

```bash
git push --force-with-lease
```

This adds a safety check intended to prevent overwriting a remote update that you have not incorporated/observed.

### Important

Neither option should be used casually on protected/shared branches.

---

## Q25. When would you use `--force-with-lease`?

Typical example:

```text
feature branch
      ↓
local rebase
      ↓
commit IDs changed
      ↓
remote feature branch must be updated
```

You may need:

```bash
git push --force-with-lease
```

It is safer than:

```bash
git push --force
```

because Git checks the expected remote state before replacing the reference.

### Interview Answer

> I would use `--force-with-lease` for an intentionally rewritten branch, such as my private feature branch after rebase. I would first fetch and confirm nobody else's work needs to be preserved.

---

# 14. Tracking / Upstream Branches

## Q26. What is an upstream/tracking branch?

A local branch can have an upstream branch configured.

For example:

```bash
git push -u origin feature/login
```

The `-u` establishes tracking between:

```text
local feature/login
        ↓
origin/feature/login
```

After that, commands such as `git pull` and `git push` can use the configured upstream by default.

Inspect it:

```bash
git branch -vv
```

### Interview Trap

`origin/feature/login` is a **remote-tracking reference** in your local repository. It is not a live network connection to the remote branch.

---

# 15. Cherry-Pick

## Q27. What is `git cherry-pick`?

Cherry-pick applies the changes introduced by one or more existing commits onto the current branch.

```bash
git cherry-pick <commit>
```

Example:

```text
main:      A---B---C
                 \
hotfix:           D
```

You can switch to another branch and cherry-pick `D`.

The result is a **new commit** on the destination branch.

### Useful for

- Production hotfixes
- Backporting a specific fix
- Moving one isolated change without merging the entire branch

### Interview Point

Cherry-pick copies the **change**, not the original commit identity.

---

## Q28. What problems can cherry-pick cause?

Because cherry-pick creates a new commit, it can lead to:

- Duplicate logical changes
- Conflicts
- More complicated history
- Future merge confusion
- Difficulty identifying that the same logical fix exists under different commit IDs

Use it deliberately rather than replacing normal branch integration.

---

# 16. Stash

## Q29. What is `git stash`?

Stash temporarily records local changes so you can switch context without committing them to the current branch.

```bash
git stash
git stash list
git stash apply
git stash pop
```

Named stash:

```bash
git stash push -m "WIP login changes"
```

### Important

Stash can include tracked working-tree changes and staged changes; untracked files require options such as:

```bash
git stash push -u
```

Ignored files require stronger options such as `-a`.

### Interview Point

> Stash is temporary work management, not a reliable long-term backup strategy.

---

## Q30. Is `git stash` a permanent backup?

No.

A stash is a Git object/reference used for temporary local work, and it can be dropped or eventually become unreachable.

For important work, a private branch and commit is generally safer:

```bash
git switch -c wip/login
git add .
git commit -m "WIP: login"
```

---

# 17. Tags

## Q31. What is a Git tag?

A tag is a reference used to identify a specific object, commonly a release commit.

```bash
git tag v1.0.0
git push origin v1.0.0
```

Annotated tag:

```bash
git tag -a v1.0.0 -m "Release 1.0.0"
```

Tags are normally treated as more stable labels than branches, although a tag reference can technically be moved.

---

## Q32. Lightweight vs annotated tag?

| Lightweight tag | Annotated tag |
|---|---|
| Simple reference | Git tag object |
| Minimal metadata | Stores tagger/message/target |
| Good for simple local labels | Better for releases |
| No tag message object | Can be signed |

For production release management, annotated tags are often preferable.

### Verify

```bash
git show v1.0.0
```

---

# 18. `.gitignore`

## Q33. What is `.gitignore`?

`.gitignore` specifies patterns for files that Git should normally ignore when they are **untracked**.

Example:

```text
.env
*.log
node_modules/
.terraform/
```

### Critical Point

If a file is already tracked, adding it to `.gitignore` does **not** stop Git from tracking it.

For example:

```bash
git rm --cached <file>
```

then commit the change.

### Interview Trap

`.gitignore` is not a secret-management system.

If a secret has already been committed, it must be treated as exposed.

---

# 19. Removing Sensitive Data

## Q34. A password was accidentally committed. Is deleting the file enough?

No.

Suppose:

```text
Commit A → password.txt
Commit B → password.txt deleted
```

The secret can still exist in the history reachable from `A`.

### Correct response

1. **Rotate/revoke the credential immediately.**
2. Determine where the secret may have been copied.
3. Remove it from history using an appropriate history-rewriting tool when required.
4. Coordinate any forced update of shared remotes.
5. Search for other copies.
6. Add secret scanning/prevention controls.

### Key Principle

> Removing a secret from the latest version does not make an exposed credential safe again. Rotate it first.

### Interview Point

Mention tools such as `git filter-repo` for history rewriting when appropriate, while recognizing that rewriting shared history requires coordination.

---

# 20. Git Log

## Q35. How do you inspect Git history effectively?

Useful commands:

```bash
git log --oneline
git log --graph --oneline --decorate --all
git log --stat
git log -p
git show <commit>
```

For branch/merge investigation:

```bash
git log --graph --oneline --decorate --all
```

This helps visualize:

- Branches
- Merge commits
- Tags
- HEAD/current references
- Divergence

### Interview Point

Use `log` to understand **history**, not just to find the latest commit.

---

# 21. Git Diff

## Q36. Explain `git diff` vs `git diff --staged`.

| Command | Shows |
|---|---|
| `git diff` | Working-tree changes not staged |
| `git diff --staged` | Changes staged in the index relative to `HEAD` |
| `git diff HEAD` | Combined working-tree + staged changes relative to `HEAD` |

Example:

```bash
git diff
git diff --staged
git diff HEAD
```

### Interview Scenario

Before committing:

```bash
git diff --staged
```

is particularly useful because it tells you what your **next commit actually contains**.

---

# 22. Git Bisect

## Q37. What is `git bisect`?

`git bisect` uses binary search to find the commit that introduced a regression.

Start:

```bash
git bisect start
git bisect bad
git bisect good <known-good-commit>
```

Git checks out candidate commits.

Test the application and tell Git:

```bash
git bisect good
```

or:

```bash
git bisect bad
```

Repeat until Git identifies the first bad commit.

Finish:

```bash
git bisect reset
```

### Why is it powerful?

If you have 1,024 candidate commits, binary search can reduce the investigation to roughly 10 test rounds instead of checking every commit sequentially.

### Interview Point

The test must be meaningful and reproducible; otherwise bisect results can be misleading.

---

# 23. Git Blame

## Q38. What is `git blame`?

`git blame` shows the commit associated with each line of a file.

```bash
git blame app.py
```

Useful for finding:

- When a line was introduced
- Which commit changed it
- Which author/committer is associated with that change

### Better Senior-Level Usage

Do not treat blame as assigning personal responsibility.

Use it as a **history investigation starting point**, then inspect the actual commit:

```bash
git show <commit>
```

---

# 24. Git Clean

## Q39. What does `git clean` do?

`git clean` removes untracked files/directories from the working tree.

Always preview first:

```bash
git clean -n
```

Then:

```bash
git clean -f
```

For directories:

```bash
git clean -fd
```

To include ignored files, stronger options such as `-x` are available.

### Warning

`git clean` can permanently remove local untracked work.

A good production-like troubleshooting habit is:

```bash
git clean -n
```

before:

```bash
git clean -f
```

---

# 25. Detached HEAD

## Q40. What is detached HEAD?

Normally:

```text
HEAD
 ↓
main
 ↓
commit C
```

In detached HEAD:

```text
HEAD
 ↓
commit C
```

HEAD points directly to a commit rather than a local branch.

Example:

```bash
git switch --detach <commit>
```

You can inspect or test historical code.

If you create useful commits, create a branch:

```bash
git switch -c recovery-branch
```

### Interview Trap

Detached HEAD does **not** mean your repository is broken. It means `HEAD` is not currently attached to a branch reference.

---

# 26. Merge vs Rebase Scenario

## Q41. Your feature branch is behind `main` and has local commits. What would you do?

First understand the remote state:

```bash
git fetch origin
git log --oneline --graph --decorate HEAD..origin/main
```

For a private feature branch, I may rebase:

```bash
git switch feature
git rebase origin/main
```

Resolve conflicts:

```bash
git add <file>
git rebase --continue
```

Then, because rebase rewrote the feature commits:

```bash
git push --force-with-lease
```

If the branch is shared and rewriting its history is undesirable, I would merge instead.

### Senior L2 Answer

> First I fetch and inspect the divergence. Then I choose merge or rebase based on whether the branch is shared and on team policy. I avoid force-pushing without verifying who may be affected.

---

# 27. Pull Request Workflow

## Q42. Describe a good Git workflow for a DevOps team.

A common workflow:

```text
Developer
   │
   │ push
   ↓
Feature Branch
   │
   ↓
Pull Request
   │
   ├── CI
   ├── Tests
   ├── Security Scans
   └── Code Review
   │
   ↓
Protected main
   │
   ↓
Release / Deployment
```

Typical controls:

- Protected main branch
- Pull Requests
- Required reviews
- Required CI checks
- Secret/security scanning
- No uncontrolled direct production changes
- Controlled release tags
- Traceability from commit → artifact → deployment

### Interview Point

The exact workflow should follow organizational policy rather than one universal branching model.

---

# 28. Branch Protection

## Q43. What is branch protection?

Branch protection is a set of repository-hosting rules that reduce unsafe changes to important branches.

Possible controls include:

- Require Pull Requests
- Require approvals
- Require successful CI checks
- Restrict force pushes
- Restrict deletion
- Require signed commits where appropriate
- Require conversation resolution
- Enforce status checks

Exact controls depend on the Git hosting platform.

### Senior L2 Answer

> Branch protection is a governance layer around Git collaboration. It does not change Git's underlying object model; it restricts risky operations at the hosting/platform level.

---

# 29. Git Hooks

## Q44. What are Git hooks?

Git hooks are scripts/programs triggered by Git lifecycle events.

Examples:

```text
pre-commit
commit-msg
pre-push
post-merge
```

Possible uses:

- Formatting
- Linting
- Commit-message validation
- Local checks
- Developer automation

### Security Limitation

Client-side hooks are not a complete enforcement mechanism because users can bypass or replace them.

Critical checks should also run in CI/server-side controls.

---

# 30. Git LFS

## Q45. What is Git LFS?

Git LFS (Large File Storage) is designed for large files such as:

- Large binaries
- Media
- Machine-learning artifacts
- Large datasets

Git stores lightweight pointer files in the normal repository while the actual large content is stored through LFS infrastructure.

### Why?

Large binary files are expensive in normal Git history because Git is optimized primarily for source-oriented versioning, not repeatedly changing large binaries.

---

# 31. Git Repository Performance

## Q46. A repository has become very large. How would you investigate?

First inspect repository/object statistics:

```bash
git count-objects -vH
```

To inspect objects:

```bash
git rev-list --objects --all
```

For deeper investigation, identify large blobs/paths and determine whether the problem is:

- Large binaries
- Generated files
- Build artifacts
- Archives
- Accidentally committed files
- Historical secrets

Potential solutions:

- Improve `.gitignore`
- Use Git LFS
- Remove generated artifacts from future commits
- Rewrite history when necessary
- Run appropriate repository maintenance

### Important

Adding a file to `.gitignore` today does not remove the file from old history.

History rewriting on a shared repository must be coordinated.

---

# 32. Git Garbage Collection

## Q47. What is `git gc`?

`git gc` performs repository maintenance such as:

- Packing objects
- Optimizing storage
- Pruning objects that are eligible to be removed according to Git's retention rules

Modern Git performs automatic maintenance in many circumstances.

### Recovery Warning

Do not casually perform aggressive cleanup when you are trying to recover recently unreachable commits.

A commit made unreachable by reset/rebase may remain recoverable for some time, but eventual cleanup can make recovery much harder or impossible.

---

# 33. Remote Troubleshooting

## Q48. `git push` fails with "non-fast-forward". What does it mean?

It generally means the remote branch has commits that your local branch does not contain, so Git will not move the remote reference forward without integrating the histories.

First:

```bash
git fetch origin
```

Then inspect:

```bash
git log HEAD..origin/main --oneline
```

Depending on the workflow:

```bash
git pull --rebase
```

or:

```bash
git merge origin/main
```

Then push again.

### Do Not Immediately Do

```bash
git push --force
```

because you may overwrite someone else's remote history.

---

# 34. Authentication Failure

## Q49. Git push suddenly asks for credentials or fails authentication. What do you check?

First inspect the remote:

```bash
git remote -v
```

Determine whether it uses:

- SSH
- HTTPS
- Credential manager
- Personal access token
- SSO/enterprise identity

### SSH checks

```bash
ssh -T git@<git-host>
ssh-add -l
```

Also inspect:

```bash
ssh -vT git@<git-host>
```

when deeper troubleshooting is required.

### HTTPS

Check whether the token/credential:

- Exists
- Has expired
- Has sufficient permissions
- Is accepted by the organization
- Requires SSO authorization

Never place credentials directly in repository URLs or scripts.

---

# 35. Reflog Recovery Scenario

## Q50. You rebased the wrong branch and lost the original commits. How do you recover?

First:

```bash
git reflog
```

Identify the `HEAD` position before the incorrect rebase.

Inspect:

```bash
git show <old-commit>
```

Create a recovery reference:

```bash
git switch -c recovery <old-commit>
```

Then compare:

```bash
git log --graph --oneline --decorate --all
```

Once the correct commits are identified, restore the intended branch carefully.

### Senior L2 Principle

> Establish a recovery reference before experimenting further. Do not make recovery harder by continuing destructive operations.

---

# 36. Senior L2 Production Scenarios

## Q51. A developer says their commit disappeared. How do you investigate?

Do not assume deletion.

Start with:

```bash
git log --all --oneline --decorate
git reflog
git branch -a
```

If necessary:

```bash
git fsck --no-reflogs
```

Possible causes:

- Branch reset
- Rebase
- Commit exists on another branch
- Commit was never pushed
- Branch deleted
- Remote branch force-pushed
- Commit became unreachable

If found, create a recovery branch immediately:

```bash
git switch -c recovery <commit>
```

---

## Q52. Someone force-pushed `main` and removed other developers' commits. What do you do?

Treat it as a history-recovery incident.

### Response

1. Stop further destructive pushes.
2. Determine the intended previous state.
3. Search local clones/reflogs for the previous tip.
4. Inspect the lost commits.
5. Create recovery references.
6. Coordinate the restoration with the team.
7. Restore the protected branch to the agreed state.
8. Prevent recurrence with branch protection and restricted force pushes.

### Key Principle

> Do not blindly force-push another "fix." First establish the correct desired history.

---

## Q53. A merge introduced a production bug. How do you identify the commit?

If you know a good commit and a bad commit:

```bash
git bisect start
git bisect bad
git bisect good <known-good-commit>
```

Run the application's test/validation at each candidate.

```bash
git bisect good
```

or:

```bash
git bisect bad
```

until Git identifies the first bad commit.

Then:

```bash
git bisect reset
```

### Senior L2 Point

Automate the test when possible:

```bash
git bisect run <test-command>
```

The test should return an appropriate success/failure exit code.

---

# 37. Git Security

## Q54. How would you secure Git usage in a DevOps environment?

I would combine technical controls and governance:

| Area | Controls |
|---|---|
| Access | Least privilege, MFA/SSO |
| Branches | Protected branches, restricted force-push |
| Review | Pull Requests, required approvals |
| CI | Tests, SAST, dependency/security scanning |
| Secrets | Secret scanning, credential rotation, secure secret storage |
| Identity | Short-lived credentials where supported |
| Integrity | Signed commits/tags where appropriate |
| Audit | Repository and platform audit logs |
| Prevention | Hooks plus server/CI enforcement |

### Important

`.gitignore` only prevents normal tracking of matching untracked files. It is not a secret vault.

---

# 38. Git + CI/CD

## Q55. How does Git integrate with Jenkins or another CI/CD system?

Typical flow:

```text
Developer
   │
   │ git push
   ↓
Remote Repository
   │
   │ webhook/event
   ↓
CI System
   │
   ├── Checkout exact revision
   ├── Build
   ├── Unit Tests
   ├── Security Scan
   ├── Package Artifact
   └── Publish
            │
            ↓
        Deployment
```

The pipeline should identify the **exact commit SHA** being built.

A strong traceability chain is:

```text
Commit SHA
    ↓
Build
    ↓
Artifact
    ↓
Deployment
```

This supports auditing and rollback.

---

# 39. Commit SHA in CI/CD

## Q56. Why is commit SHA important in CI/CD?

A branch name is mutable:

```text
main → A
```

Later:

```text
main → B
```

If a deployment record only says `main`, you cannot reliably determine which source revision produced it.

A commit SHA identifies the exact commit.

### Strong Production Traceability

```text
Commit SHA
    ↓
Source
    ↓
Build
    ↓
Artifact
    ↓
Deployment
```

This lets you answer:

- Which source produced this artifact?
- Which artifact is running?
- Which commit introduced the change?
- Which exact revision should be rolled back?

---

# 40. Senior L2 Git Command Sheet

## Repository

```bash
git init
git clone <url>
git status
git remote -v
git config --list
```

## Branches

```bash
git branch
git branch -a
git branch -vv
git switch main
git switch -c feature/test
git branch -d feature/test
```

## Changes

```bash
git diff
git diff --staged
git diff HEAD
git add <file>
git restore <file>
```

## Commits

```bash
git commit -m "message"
git commit --amend
git log --oneline
git show <commit>
```

## Remote

```bash
git fetch origin
git pull
git push
git push -u origin <branch>
git push --force-with-lease
```

## Merge

```bash
git merge <branch>
git merge --no-ff <branch>
git merge --abort
```

## Rebase

```bash
git rebase <branch>
git rebase -i HEAD~5
git rebase --continue
git rebase --skip
git rebase --abort
```

## Recovery

```bash
git reflog
git reset
git revert
git fsck
```

## Selective Changes

```bash
git cherry-pick <commit>
```

## Temporary Work

```bash
git stash
git stash list
git stash apply
git stash pop
```

## Investigation

```bash
git blame <file>
git bisect start
git log --graph --oneline --decorate --all
git count-objects -vH
```

## Cleanup

```bash
git clean -n
git clean -f
git gc
```

---

# 41. Top 20 Senior L2 Questions

Before the interview, make sure you can answer these without memorizing commands:

1. Git vs GitHub?
2. Working tree vs index vs repository?
3. What happens internally during `git commit`?
4. What are blobs, trees, commits, and tags?
5. What is a Git branch internally?
6. Merge vs rebase?
7. Fast-forward vs non-fast-forward merge?
8. How do you resolve merge/rebase conflicts?
9. Soft vs mixed vs hard reset?
10. Reset vs revert?
11. How do you recover a lost commit?
12. What is reflog and what are its limitations?
13. Fetch vs pull?
14. `--force` vs `--force-with-lease`?
15. What is cherry-pick and what problems can it cause?
16. What is detached HEAD?
17. How do you remove an accidentally committed secret?
18. How do you use `git bisect` to find a regression?
19. How do you troubleshoot a non-fast-forward push?
20. How do you maintain Git traceability in CI/CD?

---

# 42. Senior L2 Answer Strategy

When the interviewer gives you a Git problem, structure the answer like this:

```text
1. Establish current state
        ↓
   git status

2. Understand history
        ↓
   git log / git show / git reflog

3. Understand branches/remotes
        ↓
   git branch -vv
   git remote -v
   git fetch

4. Identify the exact failure
        ↓
   diff / log / bisect / object inspection

5. Choose the least-destructive fix
        ↓
   merge / rebase / revert / cherry-pick / recovery

6. Verify
        ↓
   git status
   git log
   git diff

7. Push safely
        ↓
   git push
   or
   git push --force-with-lease
```

### The Senior L2 mindset

Do not start with:

> "I will run `git reset --hard`."

Start with:

> "First I will establish the current repository state and determine whether the desired commit exists locally, remotely, or only in reflog. Then I will choose the least destructive recovery method."

---

# Final Interview Rule

For Senior L2 Git interviews, demonstrate understanding of:

```text
Git
│
├── Objects
│   ├── Blob
│   ├── Tree
│   ├── Commit
│   └── Tag
│
├── References
│   ├── Branch
│   ├── Tag
│   └── HEAD
│
├── State
│   ├── Working Tree
│   ├── Index
│   └── Repository
│
├── History Operations
│   ├── Merge
│   ├── Rebase
│   ├── Reset
│   ├── Revert
│   └── Cherry-pick
│
├── Recovery
│   ├── Reflog
│   └── fsck
│
└── Collaboration
    ├── Fetch
    ├── Pull
    ├── Push
    ├── Tracking branches
    └── Protected branches
```

A strong Senior L2 answer sounds like:

> **"First I will inspect the current Git state and establish whether the change exists locally or remotely. I will inspect the branch and commit graph before using a destructive command. Once I identify the root cause, I will choose the least destructive operation, verify the resulting history and working tree, and only then update the remote using the safest appropriate push strategy."**

That demonstrates **Git internals + troubleshooting + collaboration + risk management**, rather than simple command memorization.
