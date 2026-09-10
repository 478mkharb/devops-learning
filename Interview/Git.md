# Git Senior L2 Interview Preparation

A practical **Senior L2 Git interview guide** focused on Git internals, repository state, branching, merging, rebasing, recovery, remotes, troubleshooting, security, performance, and CI/CD.

> **Interview principle:** Do not answer only with commands. Explain **what Git state you are inspecting, why you chose a command, what risk it has, how you verify the result, and how you prevent recurrence.**

---

## How to use this guide

For each question, prepare three layers:

1. **Short interview answer** — 20–40 seconds.
2. **Technical explanation** — enough depth for follow-up questions.
3. **Production angle** — what you would actually do on a team.

The goal is not command memorization. A Senior L2 engineer should be able to reason from Git's **objects, references, working tree, index, and commit graph**.

---

# 1. Git Fundamentals

## Q1. What is Git?

### Short Interview Answer

Git is a **distributed version control system (DVCS)** that records project history as commits. A normal clone contains the repository's Git history and can create commits locally without a central server.

### Key points

- Git tracks snapshots through commits.
- Branches and tags are references to Git objects.
- Developers can work and commit offline.
- Collaboration happens through remotes.
- Git itself is different from GitHub, GitLab, or Bitbucket.

### Interview distinction

| Git | GitHub / GitLab / Bitbucket |
|---|---|
| Version-control system | Hosting/collaboration platform |
| Runs locally | Provides a hosted/service layer |
| Stores Git objects and refs | Hosts repositories and adds PRs, permissions, CI/CD, etc. |
| Works without a hosting platform | Uses Git underneath |

> **Senior point:** Git is the version-control engine; a hosting platform is an ecosystem around Git.

---

## Q2. Explain the working tree, index, and repository.

Think of Git as three important local states:

```text
Working Tree
     │
     │ git add
     ▼
Index / Staging Area
     │
     │ git commit
     ▼
Repository (.git)
```

| Area | Meaning |
|---|---|
| Working tree | Files currently checked out on disk |
| Index | Exact content selected for the next commit |
| Repository | Git object database and references |

### Example

If you edit `app.py`:

```bash
git diff
```

shows unstaged working-tree changes.

After:

```bash
git add app.py
```

inspect what the next commit contains:

```bash
git diff --staged
```

### Interview trap

`git add` does **not** mean "commit the file." It updates the index so the selected version becomes part of the next snapshot.

---

## Q3. What happens internally when you run `git commit`?

At a high level:

```text
Working Tree
     │
     │ git add
     ▼
Index
     │
     │ git commit
     ▼
Tree + Commit object
     │
     ▼
Current branch reference moves
```

Git uses the index to construct the tree representing the next snapshot. It creates or reuses the necessary objects and creates a commit object containing metadata and references.

A commit normally records:

- Parent commit(s)
- Root tree
- Author
- Committer
- Timestamps
- Commit message

The branch does not contain the file contents itself; it points to a commit.

### Useful inspection

```bash
git cat-file -t <object-id>
git cat-file -p <object-id>
git rev-parse HEAD
```

> **Senior point:** A commit is a Git object whose identity depends on its contents, including its parent relationship and tree.

---

# 2. Git Internals

## Q4. What are Git's core object types?

| Object | Purpose |
|---|---|
| **Blob** | File content |
| **Tree** | Directory structure; names, modes, and references to blobs/trees |
| **Commit** | Points to a tree and parent commit(s), plus metadata |
| **Annotated tag object** | Metadata pointing to another Git object |

### Important distinction

A blob does **not** contain the filename. The tree associates a filename/path with the blob.

An annotated tag is an actual Git object.

A lightweight tag is **only a reference**, not a tag object.

Inspect objects with:

```bash
git cat-file -t <object-id>
git cat-file -p <object-id>
```

---

## Q5. What is a Git object ID/hash?

Git uses **content-derived object IDs**. Depending on repository format, Git can use SHA-1 or SHA-256.

An object's ID is derived from its object type and content. For a commit, that content includes information such as its tree and parent references.

Therefore:

```text
Change content
     ↓
Different object content
     ↓
Different object ID
```

### Interview trap

Do not say:

> "The commit hash is just a hash of the files."

A commit ID identifies the **commit object**, not merely a file.

---

## Q6. What is `HEAD`? How are local branches and `origin/main` different?

A useful mental model is:

```text
HEAD
 │
 ▼
main
 │
 ▼
commit C
```

Here `HEAD` is attached to the local branch `main`.

A detached HEAD looks like:

```text
HEAD
 │
 ▼
commit C
```

### Local branch vs remote-tracking branch

| Reference | Meaning |
|---|---|
| `main` | Local branch reference |
| `origin/main` | Local remote-tracking reference representing the last fetched state of `main` from `origin` |
| `HEAD` | Symbolic reference to the current branch, or direct pointer in detached HEAD |

### Critical point

`origin/main` is **not a live network connection**. It changes when you fetch/pull or otherwise update the remote-tracking ref.

Useful commands:

```bash
git rev-parse HEAD
git branch -vv
git remote -v
git fetch origin
```

---

## Q7. What is a Git branch internally?

A branch is essentially a **movable reference to a commit**.

```text
A---B---C main
     \
      D---E feature
```

`main` points to `C`; `feature` points to `E`.

Creating a branch does not copy the repository:

```bash
git switch -c feature/login
```

It creates a new reference at the current commit and updates `HEAD`.

> **Senior point:** Branches are lightweight references; commits and their reachable objects contain the actual history.

---

# 3. Branching and History Integration

## Q8. What is the difference between `git checkout`, `git switch`, and `git restore`?

`git checkout` is an older multi-purpose command. It can switch branches, create branches, detach `HEAD`, and restore paths.

Modern commands separate those concepts:

```bash
git switch main
git switch -c feature/login
git restore app.py
```

| Command | Primary modern use |
|---|---|
| `git switch` | Branch switching/creation |
| `git restore` | Restore file contents |
| `git checkout` | Legacy/multi-purpose command still widely encountered |

### Interview point

The important distinction is not that `checkout` is invalid—it is valid—but that `switch` and `restore` make intent clearer.

---

## Q9. What is Git merge?

`git merge` integrates another branch into the **current branch**.

```text
A---B---C---D main
     \
      E---F feature
```

From `main`:

```bash
git switch main
git merge feature
```

If the histories have diverged, Git may create a merge commit:

```text
A---B---C---D---M
     \         /
      E---F----
```

A merge normally does not rewrite the existing commits on either branch.

---

## Q10. What is a fast-forward merge?

A fast-forward is possible when the current branch is an ancestor of the branch being merged.

Before:

```text
A---B---C main
         \
          D---E feature
```

Run:

```bash
git switch main
git merge feature
```

Git can simply move `main`:

```text
A---B---C---D---E main
```

No merge commit is created.

> **Interview point:** Fast-forward is a reference movement, not a new merge commit.

---

## Q11. Why use `git merge --no-ff`?

`--no-ff` forces a merge commit even when a fast-forward is possible:

```bash
git merge --no-ff feature
```

Conceptually:

```text
A---B---C---------M main
          \       /
           D---E
```

This can preserve an explicit integration point for a feature or release.

### Trap

`--no-ff` does not permanently preserve the branch name. It preserves a merge commit representing the integration event.

---

## Q12. What causes a merge conflict, and how do you resolve it?

A conflict occurs when Git cannot safely combine changes automatically.

Example:

```text
main:    changes line 20 to A
feature: changes line 20 to B
```

Inspect the state:

```bash
git status
```

You may see:

```text
<<<<<<< HEAD
current branch
=======
incoming branch
>>>>>>> feature
```

Resolve the file, remove conflict markers, then:

```bash
git add <resolved-file>
git commit
```

To abandon the merge:

```bash
git merge --abort
```

### Senior approach

1. Inspect the conflict context.
2. Understand both sides and the intended behavior.
3. Resolve semantically—not just syntactically.
4. Stage only reviewed files.
5. Run tests.
6. Complete the merge.

> Do not blindly use `git add .` during a complex conflict.

---

# 4. Rebase

## Q13. What is Git rebase?

Rebase changes the base of a sequence of commits by **replaying them** onto another base.

Before:

```text
A---B---C main
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
A---B---C---D'---E' feature
```

`D'` and `E'` are new commits with new IDs.

### Why do IDs change?

A commit includes its parent relationship. Changing the parent changes the commit object's content, so the resulting commit ID changes.

> **Senior point:** Rebase rewrites history even when the logical code changes are unchanged.

---

## Q14. Merge vs rebase — when should you choose each?

| Merge | Rebase |
|---|---|
| Integrates histories | Replays commits onto a new base |
| Can create merge commit | Creates rewritten commits |
| Existing commit IDs remain | Replayed commit IDs change |
| Preserves topology | Can produce a linear history |
| Good for shared history | Common for private/local feature work |

### Practical rule

Rebase private/local work when it improves clarity. Avoid rebasing commits that others are already building on unless the team explicitly coordinates the rewrite.

### Senior answer

> "I first determine whether the branch is shared. If it is private, rebase may be appropriate. If others depend on its published history, I normally avoid rewriting it and use merge or another non-destructive integration strategy."

---

## Q15. What does `git rebase -i` do?

Interactive rebase lets you modify a sequence of commits:

```bash
git rebase -i HEAD~5
```

Common operations:

| Action | Meaning |
|---|---|
| `pick` | Keep commit |
| `reword` | Keep changes, edit message |
| `edit` | Stop for manual amendment |
| `squash` | Combine with previous commit and edit message |
| `fixup` | Combine and discard this commit's message |
| `drop` | Remove commit from the rebased sequence |

Typical use: clean up local history **before sharing it**.

---

## Q16. How do you resolve a rebase conflict?

First:

```bash
git status
```

Resolve the file, then:

```bash
git add <resolved-file>
git rebase --continue
```

Repeat until complete.

Other controls:

```bash
git rebase --skip
git rebase --abort
```

### Important distinction

During a rebase, do not normally run `git commit` to finish the operation. `git rebase --continue` advances Git's rebase state machine.

---

# 5. Reset, Revert, and Recovery

## Q17. Explain `git reset --soft`, `--mixed`, and `--hard`.

`git reset` moves the current branch/`HEAD` and may also update the index and working tree.

| Mode | Branch/HEAD | Index | Tracked working tree |
|---|---|---|---|
| `--soft` | Moves | Unchanged | Unchanged |
| `--mixed` | Moves | Reset | Unchanged |
| `--hard` | Moves | Reset | Reset |

### Soft

```bash
git reset --soft HEAD~1
```

The commit is removed from the branch tip, but its changes remain staged.

### Mixed

```bash
git reset HEAD~1
```

Mixed is the default. The branch moves and the index is reset, while working-tree changes remain.

### Hard

```bash
git reset --hard HEAD~1
```

The branch, index, and tracked working tree are reset to the target state.

> **Warning:** `--hard` can discard uncommitted tracked changes.

### Senior nuance

Reset does not necessarily destroy the old commit immediately. The old commit can remain reachable through reflog or other references until it eventually becomes unreachable and eligible for cleanup.

---

## Q18. Reset vs revert — which should you use on a shared branch?

| `reset` | `revert` |
|---|---|
| Moves a reference | Creates a new commit |
| Can rewrite branch history | Preserves published history |
| Useful for private/local cleanup | Usually preferred for shared branches |
| Can affect index/worktree depending on mode | Applies inverse changes |

Example:

```text
A---B---C---D
```

```bash
git revert C
```

creates:

```text
A---B---C---D---R
```

`R` reverses the changes introduced by `C`.

### Senior answer

> "On a shared branch I normally use revert because it records the rollback without rewriting published history. I use reset when I intentionally need to move a private/local branch."

---

## Q19. How do you revert a merge commit?

A merge commit has multiple parents, so Git needs to know which parent represents the mainline.

Example:

```bash
git revert -m 1 <merge-commit>
```

`-m 1` means **use parent 1 as the mainline**; it does not mean "revert commit number 1."

### Senior caution

Before reverting a merge, inspect the graph:

```bash
git show --summary <merge-commit>
git log --graph --oneline --decorate --all
```

Choose the correct mainline based on the repository topology and intended rollback.

---

## Q20. You accidentally ran `git reset --hard`. Can you recover?

Often yes, if the desired commit has not been permanently pruned.

Start with:

```bash
git reflog
```

Find the previous branch/HEAD position:

```bash
git show <commit>
```

Create a recovery reference **before experimenting further**:

```bash
git switch -c recovery <commit>
```

Then verify:

```bash
git log --graph --oneline --decorate --all
```

### Senior recovery sequence

1. Stop destructive operations.
2. Inspect reflog.
3. Identify the correct old tip.
4. Create a recovery branch.
5. Verify commits/files.
6. Restore the intended branch carefully.

---

## Q21. What is reflog, and what are its limitations?

Reflog records **local movements of references**, such as `HEAD` and local branches.

```bash
git reflog
git reflog show main
```

It is useful after:

- Reset
- Rebase
- Accidental branch movement
- Detached HEAD work
- Recovery of a previously referenced commit

### Limitations

- It is primarily local.
- It is not a remote backup.
- Entries can expire.
- Eventually unreachable objects may be pruned.

> **Interview trap:** "I can always recover it with reflog" is incorrect.

---

## Q22. A commit "disappeared." How do you investigate?

Do not assume it was deleted.

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
- Branch deletion
- Commit exists on another branch
- Commit was never pushed
- Remote history was force-pushed
- Commit became unreachable

If you find the commit:

```bash
git switch -c recovery <commit>
```

### Senior principle

Create a recovery reference first. Do not make recovery harder by continuing destructive operations.

---

# 6. Remote Repositories

## Q23. What is the difference between `git fetch` and `git pull`?

`git fetch` downloads remote updates and updates remote-tracking references without integrating those changes into your current branch.

```bash
git fetch origin
```

`git pull` generally means:

```text
fetch
 +
integration
```

The integration may be merge or rebase depending on options/configuration.

### Safer investigation

```bash
git fetch origin
git log HEAD..origin/main --oneline
git diff HEAD..origin/main
```

Then choose the appropriate integration strategy.

> **Senior point:** Fetch separates **information gathering** from **history integration**.

---

## Q24. What is `origin`? What is `upstream`?

`origin` is a conventional remote name automatically assigned when cloning. It has no special protocol meaning.

```bash
git remote -v
```

You can have multiple remotes:

```bash
git remote add upstream <url>
```

A common fork workflow is:

```text
origin   → your fork
upstream → original project
```

Remote names are configurable labels.

---

## Q25. What is an upstream/tracking branch?

A local branch can track another branch, commonly a remote-tracking branch:

```bash
git push -u origin feature/login
```

This establishes an upstream relationship.

Inspect it:

```bash
git branch -vv
```

Then Git can infer the default remote branch for commands such as push/pull in many normal workflows.

### Important distinction

`feature/login` and `origin/feature/login` are separate local references.

---

## Q26. What is the difference between normal push, force push, and force-with-lease?

Normal push generally requires a fast-forward-compatible update.

```bash
git push
```

Force push permits replacing the remote branch reference:

```bash
git push --force
```

Safer for intentional rewrites:

```bash
git push --force-with-lease
```

| Method | Risk |
|---|---|
| `git push` | Lowest for normal collaboration |
| `git push --force-with-lease` | Controlled rewrite with a remote-state expectation check |
| `git push --force` | Can overwrite remote history without that safety check |

### Important nuance

`--force-with-lease` is **safer, not magically safe**. Fetch first and verify that you are not discarding someone else's work.

---

## Q27. A feature branch was rebased locally. How do you update the remote safely?

Typical flow:

```bash
git fetch origin
git switch feature
git rebase origin/main
```

Resolve conflicts and test.

Because the rebase changed commit IDs:

```bash
git push --force-with-lease origin feature
```

### Senior checklist

- Confirm the branch is intended to be rewritten.
- Confirm nobody else has pushed work that must be preserved.
- Fetch current remote state.
- Review the resulting graph.
- Prefer `--force-with-lease` over `--force`.
- Never normalize force-pushing protected/shared branches.

---

## Q28. `git push` says "non-fast-forward." What does it mean?

It generally means the remote branch contains commits that your local branch does not contain.

First:

```bash
git fetch origin
git log HEAD..origin/main --oneline
```

Then choose the correct integration strategy:

```bash
git pull --rebase
```

or:

```bash
git merge origin/main
```

Then push again.

### Do not immediately do this

```bash
git push --force
```

You may overwrite someone else's work.

---

# 7. Selective and Temporary Changes

## Q29. What is `git cherry-pick`? When would you use it?

Cherry-pick applies the changes introduced by an existing commit onto the current branch:

```bash
git cherry-pick <commit>
```

It creates a **new commit** on the destination branch.

### Common uses

- Production hotfix
- Backporting a fix to a release branch
- Moving one isolated change without merging an entire feature branch

### Risks

- Duplicate logical changes
- Conflicts
- More complicated history
- Future merge confusion

> **Interview point:** Cherry-pick copies the **change**, not the original commit identity.

---

## Q30. What is `git stash`? Is it a backup?

Stash temporarily records local changes so you can change context without committing them to the current branch.

```bash
git stash
git stash list
git stash apply
git stash pop
```

For untracked files:

```bash
git stash push -u
```

For ignored files too:

```bash
git stash push -a
```

### Is it a permanent backup?

No.

A stash is temporary local work management. It can be dropped or eventually become unreachable.

For important work, a private branch and commit is generally safer:

```bash
git switch -c wip/login
git add .
git commit -m "WIP: login"
```

---

# 8. Tags and Repository Hygiene

## Q31. Lightweight vs annotated tags?

| Lightweight tag | Annotated tag |
|---|---|
| Reference only | Actual Git tag object |
| Minimal metadata | Tagger, message, target, optional signature |
| Good for simple labels | Better suited to releases |
| No tag object/message | Can be signed |

Examples:

```bash
git tag v1.0.0
```

and:

```bash
git tag -a v1.0.0 -m "Release 1.0.0"
```

For production release management, annotated tags are often preferable.

Inspect:

```bash
git show v1.0.0
```

---

## Q32. What does `.gitignore` do, and what does it NOT do?

`.gitignore` defines patterns for files Git should normally ignore when they are **untracked**.

Example:

```text
.env
*.log
node_modules/
.terraform/
```

### Critical limitation

If a file is already tracked, adding it to `.gitignore` does not stop tracking it.

Use:

```bash
git rm --cached <file>
```

then commit the change.

### Security trap

`.gitignore` is not a secret-management system.

If a secret has already been committed, treat the credential as exposed.

---

## Q33. A password/API key was committed. What is the correct response?

Deleting the file in a later commit is not sufficient because the old commit remains in history.

Correct sequence:

1. **Rotate/revoke the credential immediately.**
2. Determine where it may have been copied.
3. Remove it from history if required.
4. Coordinate any history rewrite/forced update.
5. Search other branches, clones, artifacts, logs, and caches as appropriate.
6. Add preventive secret scanning and secure secret storage.

For history rewriting, `git filter-repo` is a modern tool to consider.

### Core principle

> Remove the secret from Git history, but rotate the credential first. History cleanup does not make an already exposed credential safe.

---

# 9. Investigation and Diagnostics

## Q34. How do you inspect Git history effectively?

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
- HEAD
- Divergence
- Topology

### Senior point

Use `git log` to understand **history and causality**, not merely to find the latest commit.

---

## Q35. Explain `git diff`, `git diff --staged`, and `git diff HEAD`.

| Command | Comparison |
|---|---|
| `git diff` | Working tree vs index |
| `git diff --staged` | Index vs `HEAD` |
| `git diff HEAD` | Working tree + index vs `HEAD` |

Typical pre-commit review:

```bash
git diff --staged
```

This tells you what the next commit actually contains.

Remote comparison:

```bash
git diff HEAD..origin/main
```

after fetching current remote state.

---

## Q36. What is `git bisect` and why is it useful?

`git bisect` uses binary search to identify the commit that introduced a regression.

Start:

```bash
git bisect start
git bisect bad
git bisect good <known-good-commit>
```

Git checks out candidate commits. Test each candidate and classify it:

```bash
git bisect good
```

or:

```bash
git bisect bad
```

Finish:

```bash
git bisect reset
```

If there are roughly 1,024 candidate commits, binary search needs about 10 classification rounds rather than a linear search through all commits.

### Automation

```bash
git bisect run <test-command>
```

The command must return meaningful exit codes.

### Interview trap

Bisect is only as reliable as the **good/bad test**. A flaky or environment-dependent test can produce misleading results.

---

## Q37. What is `git blame` and how should a Senior engineer use it?

`git blame` associates each line with the commit that last changed it:

```bash
git blame app.py
```

Use it to investigate:

- When a line changed
- Which commit introduced it
- Which author/committer is associated with the change

Then inspect the actual commit:

```bash
git show <commit>
```

### Senior principle

Do not use blame as a mechanism for assigning personal fault. Use it as a **history investigation starting point**.

---

## Q38. What does `git clean` do?

`git clean` removes untracked files/directories from the working tree.

Always preview:

```bash
git clean -n
```

Then, if appropriate:

```bash
git clean -f
```

For directories:

```bash
git clean -fd
```

To include ignored files, options such as `-x` exist.

> **Warning:** `git clean` can permanently remove local untracked work.

---

## Q39. What is detached HEAD?

Normally:

```text
HEAD
 ↓
main
 ↓
commit C
```

Detached HEAD:

```text
HEAD
 ↓
commit C
```

`HEAD` points directly to a commit rather than a local branch.

Example:

```bash
git switch --detach <commit>
```

This is useful for testing historical code.

If you make valuable commits while detached, create a branch:

```bash
git switch -c recovery-branch
```

### Trap

Detached HEAD does **not** mean the repository is broken.

---

# 10. Senior Production Scenarios

## Q40. Your feature branch is behind `main` and has local commits. What do you do?

First establish the current remote state:

```bash
git fetch origin
git log --graph --oneline --decorate --all
```

Then inspect the divergence:

```bash
git log HEAD..origin/main --oneline
git log origin/main..HEAD --oneline
```

For private feature work:

```bash
git switch feature
git rebase origin/main
```

Resolve/test, then:

```bash
git push --force-with-lease
```

If the branch is shared, prefer a non-rewriting integration strategy according to team policy.

### Senior answer

> "First I fetch and inspect the divergence. Then I choose merge or rebase based on branch ownership, publication status, and team policy. I don't force-push until I have verified the remote state and impact."

---

## Q41. You rebased the wrong branch and need the original history back. What do you do?

Use reflog:

```bash
git reflog
```

Find the position before the incorrect rebase:

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

Only after identifying the correct history should you restore the intended branch.

> **Senior principle:** Establish a recovery reference before performing further recovery operations.

---

## Q42. Someone force-pushed `main` and removed other developers' commits. What is your response?

Treat it as a **history-recovery incident**, not simply another Git command problem.

### Response

1. Stop further destructive pushes.
2. Determine the intended previous `main` tip.
3. Search local clones and reflogs for the previous tip.
4. Inspect the lost commits.
5. Create recovery references.
6. Coordinate restoration with the team.
7. Restore the protected branch to the agreed state.
8. Investigate how force-push protection failed or was bypassed.
9. Restrict force pushes and improve branch protection.

### Trap

Do not blindly force-push another "fix." First establish the desired canonical history.

---

## Q43. A production regression was introduced somewhere in the last 200 commits. How do you find it efficiently?

Use bisect.

```bash
git bisect start
git bisect bad
git bisect good <known-good-commit>
```

Run the regression test at each candidate.

For repeatable tests:

```bash
git bisect run ./test-regression.sh
```

Once identified:

```bash
git bisect reset
git show <bad-commit>
```

Then decide whether the correct remediation is revert, forward fix, or another controlled rollback.

### Senior point

The hard part is not running `bisect`; it is defining a reliable **binary good/bad predicate**.

---

## Q44. A merge introduced a bug, but the merge commit itself contains no obvious bad line. How do you reason about it?

Do not assume the merge commit's own textual diff explains the whole regression.

Investigate:

```bash
git show --cc <merge-commit>
git log --graph --oneline --decorate --all
```

Then inspect the commits brought into the branch.

A merge changes the resulting tree based on both parents and conflict-resolution decisions. The defect may therefore originate from:

- A feature commit
- An incorrect conflict resolution
- An interaction between two changes
- A dependency/configuration change

Use bisect or targeted testing against the relevant history when needed.

---

## Q45. How would you investigate a repository that has become very large?

First determine whether the growth is from normal history or large objects:

```bash
git count-objects -vH
git rev-list --objects --all
```

Investigate large:

- Binaries
- Archives
- Generated files
- Build artifacts
- Datasets
- Accidentally committed files
- Historical secrets

Potential remedies:

| Problem | Possible response |
|---|---|
| Generated artifacts | Stop tracking; improve `.gitignore` |
| Large binaries | Consider Git LFS |
| Bad historical content | History rewrite when justified |
| Many loose objects | Repository maintenance |
| Old/unreachable data | Normal Git maintenance/pruning policy |

### Important

Adding a file to `.gitignore` today does **not** remove it from old history.

---

## Q46. What does `git gc` do, and why can it matter during recovery?

Git maintenance can:

- Pack objects
- Optimize storage
- Remove/prune objects eligible for cleanup
- Reduce repository overhead

Modern Git can perform maintenance automatically.

### Recovery warning

If you are recovering recently unreachable commits, do not casually run aggressive cleanup. An unreachable object may remain recoverable for a period, but eventual pruning can make recovery impossible.

> **Senior point:** Repository maintenance and recovery are related because reachability determines whether old objects remain available.

---

# 11. Remote and Authentication Troubleshooting

## Q47. Git push suddenly fails authentication. How do you troubleshoot?

Start with:

```bash
git remote -v
```

Determine the transport:

- SSH
- HTTPS
- Credential manager
- Personal access token
- SSO/enterprise identity

### SSH

```bash
ssh -T git@<git-host>
ssh-add -l
ssh -vT git@<git-host>
```

Check:

- Correct key
- Agent state
- Host/key configuration
- Repository permissions
- Organization SSO requirements

### HTTPS

Check:

- Token exists
- Token has not expired
- Token has required permissions
- Organization policy allows it
- SSO authorization is satisfied
- Credential manager is not returning stale credentials

### Security rule

Never put credentials directly in repository URLs, shell scripts, or source code.

---

## Q48. A developer can clone but cannot push. What does that tell you?

It suggests that **read access is working but write authorization may not be**—although transport, authentication, branch policy, or repository configuration can still be involved.

Investigate:

```bash
git remote -v
git ls-remote origin
```

Then verify:

- Identity/authentication
- Repository permissions
- Target branch permissions
- Protected branch rules
- Required PR workflow
- SSO/token scope
- Correct remote URL

### Senior point

Separate the problem into:

```text
Can I reach the remote?
        ↓
Can I authenticate?
        ↓
Can I read?
        ↓
Can I write?
        ↓
Is this branch allowed to be updated?
```

---

# 12. Collaboration and Governance

## Q49. Describe a good Git workflow for a DevOps team.

A common controlled flow is:

```text
Developer
   │
   │ push
   ▼
Feature Branch
   │
   ▼
Pull Request
   │
   ├── CI
   ├── Tests
   ├── Security Scans
   └── Code Review
   │
   ▼
Protected main
   │
   ▼
Release / Deployment
```

Typical controls:

- Protected main branch
- Pull Requests
- Required reviews
- Required CI checks
- Secret/security scanning
- Restricted force-push
- Controlled release tags
- Traceability from commit → artifact → deployment

### Important

There is no single universal branching model. The workflow should match organizational release and deployment policy.

---

## Q50. What is branch protection?

Branch protection is a **hosting/platform governance layer** that reduces unsafe changes to important branches.

Possible controls:

- Require Pull Requests
- Require approvals
- Require successful CI checks
- Restrict force pushes
- Restrict branch deletion
- Require signed commits where appropriate
- Require status checks
- Require conversation resolution

### Important distinction

Branch protection does not change Git's underlying object model. It restricts operations at the hosting/platform layer.

---

## Q51. What are Git hooks, and are they sufficient for security enforcement?

Hooks are programs triggered by Git lifecycle events.

Examples:

```text
pre-commit
commit-msg
pre-push
post-merge
```

Uses include:

- Formatting
- Linting
- Commit-message validation
- Local automation
- Pre-push tests

### Security limitation

Client-side hooks are not a complete enforcement mechanism because users can bypass or replace them.

Critical controls should also be enforced through:

- CI
- Server/platform rules
- Protected branches
- Central security tooling

---

# 13. Git LFS and Performance

## Q52. What is Git LFS and when would you use it?

Git LFS (Large File Storage) is designed for large files such as:

- Large binaries
- Media
- Machine-learning artifacts
- Large datasets

Normal Git stores a lightweight pointer in the repository while the actual large content is stored in LFS infrastructure.

### Why?

Repeatedly changing large binaries can make normal Git history expensive in storage and transfer.

Use LFS when the files genuinely need versioning but are poorly suited to ordinary Git object storage.

---

## Q53. How would you improve Git performance in a large repository?

First measure instead of guessing.

Useful areas to investigate:

- Repository/object size
- Large blobs
- Generated artifacts
- Huge histories
- Large working trees
- Unnecessary files
- Remote transfer volume
- CI clone/fetch behavior
- Repository maintenance

Useful commands include:

```bash
git count-objects -vH
git rev-list --objects --all
git maintenance run
```

Depending on repository size and workflow, consider:

- Better `.gitignore`
- Git LFS for suitable large files
- Removing generated artifacts from version control
- History cleanup when justified
- Appropriate Git maintenance
- CI strategies such as shallow/partial/sparse workflows where compatible with the build

### Senior principle

Do not use history rewriting as the first response to every large repository. Identify the actual source of growth first.

---

# 14. Git Security

## Q54. How would you secure Git usage in a DevOps environment?

Use layered controls:

| Area | Controls |
|---|---|
| Access | Least privilege, MFA/SSO |
| Branches | Protected branches, restricted force-push |
| Review | Pull Requests, required approvals |
| CI | Tests, SAST, dependency/security scanning |
| Secrets | Secret scanning, rotation, secure secret storage |
| Identity | Short-lived credentials where supported |
| Integrity | Signed commits/tags where appropriate |
| Audit | Repository/platform audit logs |
| Prevention | Hooks plus CI/server enforcement |

### Important distinction

Signed commits/tags can help establish authenticity/integrity of the signed object. They do **not** prove that the code is safe.

`.gitignore` also does not protect secrets.

---

# 15. Git + CI/CD

## Q55. How should Git integrate with Jenkins/CI/CD, and why is the commit SHA critical?

A typical flow is:

```text
Developer
   │
   │ push
   ▼
Remote Repository
   │
   │ webhook/event
   ▼
CI System
   │
   ├── Checkout exact revision
   ├── Build
   ├── Unit Tests
   ├── Security Scan
   ├── Package Artifact
   └── Publish
            │
            ▼
        Deployment
```

The pipeline should build a **specific revision**, not an ambiguous moving branch.

A branch name is mutable:

```text
main → A
```

Later:

```text
main → B
```

A deployment record that only says `main` cannot reliably identify which source revision produced it.

A commit SHA identifies the exact commit.

### Strong production traceability

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

This supports:

- Auditing
- Reproducibility
- Rollback
- Incident investigation

A good deployment record should let you answer:

- Which source produced this artifact?
- Which artifact is running?
- Which commit introduced the change?
- Which exact revision should be rolled back?

> **Senior L2 point:** Git integration is not just "Jenkins checks out the repo." The important engineering property is deterministic traceability from **immutable source revision → build → immutable artifact → deployment**.

---

# 16. Senior L2 Command Sheet

## Repository / state

```bash
git status
git init
git clone <url>
git remote -v
git config --list
git rev-parse HEAD
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

## Changes / index

```bash
git diff
git diff --staged
git diff HEAD
git add <file>
git restore <file>
```

## Commits / history

```bash
git commit -m "message"
git commit --amend
git log --oneline
git log --graph --oneline --decorate --all
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
git fsck --no-reflogs
```

## Selective / temporary work

```bash
git cherry-pick <commit>
git stash
git stash list
git stash apply
git stash pop
```

## Investigation / maintenance

```bash
git blame <file>
git bisect start
git bisect run <test-command>
git count-objects -vH
git maintenance run
git clean -n
git clean -f
```

---

# 17. Top 20 Senior L2 Questions

Before an interview, be able to answer these without relying on command memorization:

1. **What is the difference between Git and GitHub/GitLab?**
2. **Explain working tree vs index vs repository.**
3. **What happens internally during `git commit`?**
4. **What are blob, tree, commit, and annotated tag objects?**
5. **What is a branch internally?**
6. **What is `HEAD` and how does it relate to a branch?**
7. **What is `origin/main` actually representing?**
8. **Merge vs rebase—when would you choose each?**
9. **What causes a merge/rebase conflict and how do you resolve it safely?**
10. **Soft vs mixed vs hard reset?**
11. **Reset vs revert on a shared branch?**
12. **How do you revert a merge commit with `-m`?**
13. **How do you recover a lost commit using reflog?**
14. **Fetch vs pull, and why is fetch useful during troubleshooting?**
15. **`--force` vs `--force-with-lease`?**
16. **What is cherry-pick and what problems can it create?**
17. **How do you remove a committed secret correctly?**
18. **How do you use `git bisect` to find a production regression?**
19. **How do you troubleshoot a non-fast-forward push or authentication failure?**
20. **How do you guarantee commit → artifact → deployment traceability in CI/CD?**

---

# 18. Senior L2 Answer Strategy

When an interviewer gives you a Git incident, structure the response like this:

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
   status / log / diff / tests

7. Update the remote safely
        ↓
   git push
   or
   git push --force-with-lease
```

### A strong Senior L2 answer sounds like:

> **"First I will establish the current repository state and determine whether the desired commit exists locally, remotely, or only in reflog. I will inspect the branch and commit graph before using a destructive command. Once I identify the root cause, I will choose the least-destructive operation, verify the resulting history and working tree, run the relevant validation, and only then update the remote using the safest appropriate push strategy."**

---

# 19. High-Value Interview Traps

These are common places where a technically correct command can still produce a weak interview answer.

| Trap | Correct Senior-level understanding |
|---|---|
| "GitHub is Git" | Git is the VCS; GitHub is a hosting/collaboration platform |
| "A branch contains the files" | A branch is a movable reference to a commit |
| "`origin/main` is the remote branch itself" | It is a local remote-tracking reference updated by fetch/pull |
| "`git add` commits a file" | It updates the index for the next commit |
| "Rebase changes only the branch pointer" | Rebase replays commits and creates new commit IDs |
| "`reset` deletes commits immediately" | It moves references; old commits may remain recoverable |
| "Reflog is a remote backup" | Reflog is primarily local and can expire |
| "`--force-with-lease` is completely safe" | It is safer, but remote state and collaboration still need verification |
| "`git revert <merge>` is enough" | A merge revert normally needs the correct mainline parent via `-m` |
| "`.gitignore` removes a tracked secret" | It does not; tracked content requires untracking/history cleanup |
| "Deleting a secret file fixes exposure" | Rotate/revoke first; old history may still contain it |
| "`git clean` is harmless cleanup" | It can permanently delete untracked work |
| "Detached HEAD means broken Git" | HEAD simply points directly to a commit |
| "Blame tells who is responsible" | It is a history investigation tool |
| "Hooks enforce security" | Client hooks can be bypassed; use CI/server controls |
| "Signed commits mean safe code" | Signatures help with authenticity/integrity, not code safety |
| "A branch name identifies a deployment" | Use the exact commit SHA and artifact identity |
| "Bisect always finds the bug" | It requires a reliable, reproducible good/bad test |

---

# 20. Final Senior L2 Checklist

A Senior L2 Git engineer should be comfortable explaining:

```text
Git
│
├── State
│   ├── Working Tree
│   ├── Index
│   └── Repository
│
├── Objects
│   ├── Blob
│   ├── Tree
│   ├── Commit
│   └── Annotated Tag
│
├── References
│   ├── Branch
│   ├── Tag
│   ├── HEAD
│   └── Remote-tracking refs
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
│   ├── fsck
│   └── Recovery branches
│
├── Collaboration
│   ├── Fetch
│   ├── Pull
│   ├── Push
│   ├── Tracking branches
│   ├── Force-with-lease
│   └── Protected branches
│
├── Investigation
│   ├── log
│   ├── diff
│   ├── bisect
│   ├── blame
│   └── object inspection
│
├── Security
│   ├── Secrets
│   ├── Signing
│   ├── Access control
│   └── Secret scanning
│
└── CI/CD
    ├── Exact commit SHA
    ├── Reproducible build
    ├── Artifact identity
    └── Deployment traceability
```

## Final Interview Rule

Do not begin a production Git answer with:

> "I will run `git reset --hard`."

Begin with:

> **"First I will establish the current state, inspect the history and references, determine what is recoverable and what is shared, and then choose the least-destructive operation. I will verify the result before changing the remote."**

That demonstrates **Git internals + troubleshooting + collaboration + risk management**, which is what distinguishes a Senior L2 answer from command memorization.
