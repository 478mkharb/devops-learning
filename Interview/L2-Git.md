# Git Senior L2 Interview Preparation

A practical **Senior L2 Git interview preparation guide** focused on
real-world DevOps workflows, troubleshooting, branching, merging,
rebasing, recovery, and Git internals.

------------------------------------------------------------------------

## How to Use This Guide

For Senior L2 interviews, do not answer only with commands.

Use this pattern:

``` text
Problem
   ↓
Understand current Git state
   ↓
Run the appropriate Git command
   ↓
Identify the cause
   ↓
Apply the least-risk fix
   ↓
Verify
   ↓
Prevent recurrence
```

Useful commands should be explained along with **why** you are using
them.

------------------------------------------------------------------------

# 1. Git Fundamentals

## Q1. What is Git?

### Answer

Git is a distributed version control system used to track changes in
source code and other files.

Unlike a centralized VCS, each Git clone contains its own repository
history.

Main benefits:

-   Distributed development
-   Branching
-   Merging
-   Rewriting local history
-   Collaboration
-   Version tracking
-   Offline commits
-   Recovery using Git objects and reflog

------------------------------------------------------------------------

## Q2. What is the difference between Git and GitHub?

### Answer

**Git** is the version control system.

**GitHub** is a platform that hosts Git repositories and provides
collaboration features such as:

-   Pull Requests
-   Code Reviews
-   Issues
-   Actions/CI
-   Repository permissions
-   Branch protection

Git can be used without GitHub.

------------------------------------------------------------------------

# 2. Git Architecture

## Q3. Explain the Git working tree, staging area and repository.

### Answer

Git has three important areas:

``` text
Working Tree
     |
     | git add
     v
Staging Area / Index
     |
     | git commit
     v
Local Repository
```

### Working Tree

Files currently checked out and being modified.

### Staging Area

The proposed content for the next commit.

### Local Repository

Contains commits, branches, tags and Git objects.

Important commands:

``` bash
git status
git add <file>
git commit -m "message"
```

------------------------------------------------------------------------

## Q4. What happens internally when you run `git commit`?

### Answer

Git takes the content represented by the staging area, creates or
references the necessary Git objects, creates a commit object containing
metadata and the tree representing the staged content, and moves the
current branch reference to the new commit.

Conceptually:

``` text
Files
  |
  v
Blob objects
  |
  v
Tree
  |
  v
Commit
  |
  v
Branch reference
```

A commit contains information such as:

-   Author
-   Committer
-   Message
-   Parent commit(s)
-   Tree reference

------------------------------------------------------------------------

# 3. Git Objects

## Q5. What are Git's main object types?

### Answer

The primary Git object types are:

1.  Blob
2.  Tree
3.  Commit
4.  Tag

### Blob

Stores file content.

### Tree

Represents directory structure and references blobs/other trees.

### Commit

Points to a tree and parent commit(s), along with metadata.

### Tag

An object that can annotate another Git object, commonly used for signed
or annotated release tags.

------------------------------------------------------------------------

## Q6. What is a Git hash?

### Answer

Git identifies objects using content-derived object IDs.

Modern Git repositories can use SHA-1 or SHA-256 depending on repository
configuration.

The hash allows Git to identify objects and detect content changes.

Example:

``` bash
git log --oneline
git cat-file -t <object-id>
git cat-file -p <object-id>
```

------------------------------------------------------------------------

# 4. Branching

## Q7. What is a Git branch?

### Answer

A branch is essentially a movable reference pointing to a commit.

For example:

``` text
A---B---C   main
         \
          D---E   feature
```

Creating a branch does not copy the complete repository.

``` bash
git switch -c feature/login
```

------------------------------------------------------------------------

## Q8. What is the difference between `git checkout` and `git switch`?

### Answer

`git checkout` is an older, multi-purpose command that can switch
branches and also restore files.

`git switch` was introduced specifically for branch switching and is
clearer for modern workflows.

Examples:

``` bash
git switch main
git switch -c feature/login
```

For restoring files:

``` bash
git restore <file>
```

------------------------------------------------------------------------

# 5. Merge

## Q9. What is Git merge?

### Answer

Merge combines histories from different branches.

Example:

``` text
A---B---C---D   main
     \
      E---F     feature
```

After merging:

``` text
A---B---C---D---M   main
     \         /
      E---F----
```

`M` is a merge commit in a normal non-fast-forward merge.

------------------------------------------------------------------------

## Q10. What is a fast-forward merge?

### Answer

A fast-forward occurs when the target branch has no new commits since
the source branch diverged.

Example:

``` text
A---B---C   main
         \
          D---E   feature
```

If main has not moved, Git can simply move the main reference:

``` text
A---B---C---D---E   main
```

No merge commit is required.

------------------------------------------------------------------------

## Q11. What is `--no-ff`?

### Answer

`--no-ff` forces Git to create a merge commit even when a fast-forward
merge is possible.

``` bash
git merge --no-ff feature
```

This can preserve a visible branch integration point in project history.

------------------------------------------------------------------------

# 6. Merge Conflicts

## Q12. What causes a merge conflict?

### Answer

A conflict occurs when Git cannot automatically reconcile changes
between branches.

For example, two branches modify the same part of a file differently.

Check:

``` bash
git status
```

Git marks the conflicting file.

Conflict markers look like:

``` text
<<<<<<< HEAD
current branch content
=======
incoming branch content
>>>>>>> feature
```

Resolve the file, then:

``` bash
git add <file>
git commit
```

For a merge, you can abort before completing it:

``` bash
git merge --abort
```

------------------------------------------------------------------------

# 7. Rebase

## Q13. What is Git rebase?

### Answer

Rebase moves or reapplies commits onto another base commit.

Example:

``` text
A---B---C   main
     \
      D---E   feature
```

After:

``` bash
git switch feature
git rebase main
```

Conceptually:

``` text
A---B---C---D'---E'   feature
```

The commits are recreated, so their commit IDs change.

------------------------------------------------------------------------

## Q14. Merge vs rebase?

### Answer

  Merge                               Rebase
  ----------------------------------- --------------------------------------
  Preserves branch topology           Creates a linear history
  Can create merge commits            Replays commits
  Does not rewrite existing commits   Rewrites commit IDs
  Safe for shared history             Avoid rebasing shared public history

Typical rule:

> Rebase private/local work; avoid rewriting history that other people
> are already using.

------------------------------------------------------------------------

## Q15. What does `git rebase -i` do?

### Answer

Interactive rebase allows you to modify a sequence of commits.

Common operations:

``` text
pick
reword
edit
squash
fixup
drop
```

Example:

``` bash
git rebase -i HEAD~5
```

Useful for cleaning up local commits before creating a Pull Request.

------------------------------------------------------------------------

# 8. Rebase Conflict

## Q16. You are rebasing and get conflicts. What do you do?

### Answer

First:

``` bash
git status
```

Resolve the conflicting files.

Then:

``` bash
git add <resolved-file>
git rebase --continue
```

Repeat until complete.

If the rebase should be abandoned:

``` bash
git rebase --abort
```

Important:

Do not blindly run `git add .` without checking what was changed.

------------------------------------------------------------------------

# 9. Reset

## Q17. Explain `git reset --soft`, `--mixed`, and `--hard`.

### Answer

### Soft

Moves HEAD/branch reference but keeps changes staged.

``` bash
git reset --soft HEAD~1
```

### Mixed

Default reset mode.

Moves HEAD and resets the index, but keeps working-tree changes.

``` bash
git reset HEAD~1
```

### Hard

Moves HEAD and resets both index and working tree.

``` bash
git reset --hard HEAD~1
```

**Danger:** `--hard` can discard local working-tree changes.

------------------------------------------------------------------------

# 10. Revert

## Q18. What is the difference between reset and revert?

### Answer

`git reset` moves a branch reference and can rewrite local history.

`git revert` creates a new commit that reverses the effect of an earlier
commit.

For a shared branch such as `main`, revert is usually safer.

``` bash
git revert <commit>
```

Example:

``` text
A---B---C---D   main

git revert C

A---B---C---D---R
```

`R` reverses the changes introduced by C.

------------------------------------------------------------------------

# 11. Recovering Lost Commits

## Q19. You accidentally ran `git reset --hard` and lost a commit. Can it be recovered?

### Answer

Often yes, if the relevant Git objects have not been garbage-collected.

Use:

``` bash
git reflog
```

Find the previous HEAD:

``` text
HEAD@{1}
HEAD@{2}
```

Then inspect:

``` bash
git show <commit>
```

Recover by creating a branch:

``` bash
git switch -c recovery <commit>
```

`reflog` is one of the most important recovery tools in Git.

------------------------------------------------------------------------

## Q20. What is reflog?

### Answer

Reflog records movements of references such as HEAD and local branches.

Example:

``` bash
git reflog
```

It can help recover commits after:

-   Reset
-   Rebase
-   Branch movement
-   Accidental checkout/reset operations

Important:

Reflog is primarily local and is not a replacement for a remote backup.

------------------------------------------------------------------------

# 12. Remote Repositories

## Q21. What is the difference between `git fetch` and `git pull`?

### Answer

`git fetch` downloads remote updates without integrating them into the
current local branch.

``` bash
git fetch origin
```

`git pull` normally performs a fetch followed by an integration
operation, typically merge or rebase depending on configuration/options.

Conceptually:

``` text
git pull
   =
git fetch
   +
merge/rebase
```

------------------------------------------------------------------------

## Q22. Why is `git fetch` safer for inspection?

### Answer

It updates remote-tracking references such as:

``` text
origin/main
```

without changing the current branch's working tree or local branch
history.

You can inspect first:

``` bash
git fetch origin
git log HEAD..origin/main --oneline
```

Then decide whether to merge or rebase.

------------------------------------------------------------------------

## Q23. What is `origin`?

### Answer

`origin` is simply the conventional default name for a remote
repository.

Check:

``` bash
git remote -v
```

A repository can have multiple remotes:

``` bash
git remote add upstream <url>
```

------------------------------------------------------------------------

# 13. Push

## Q24. What is the difference between `git push` and `git push --force`?

### Answer

Normal push updates the remote branch only when Git can safely move it
forward.

Force push allows the remote branch reference to be rewritten.

``` bash
git push --force
```

This can overwrite commits on the remote branch.

A safer option is:

``` bash
git push --force-with-lease
```

`--force-with-lease` checks that the remote branch is still at the
expected state before overwriting it.

------------------------------------------------------------------------

## Q25. When would you use `--force-with-lease`?

### Answer

Common case:

You rebased your private feature branch and need to update its remote
branch.

``` bash
git push --force-with-lease
```

It is safer than `--force` because it protects against overwriting
remote changes that you have not seen.

------------------------------------------------------------------------

# 14. Tracking Branches

## Q26. What is an upstream/tracking branch?

### Answer

A local branch can track a remote branch.

Example:

``` bash
git push -u origin feature/login
```

The `-u` establishes the upstream relationship.

After that, commands such as:

``` bash
git pull
git push
```

can use the configured upstream automatically.

Check:

``` bash
git branch -vv
```

------------------------------------------------------------------------

# 15. Cherry-Pick

## Q27. What is `git cherry-pick`?

### Answer

Cherry-pick applies the changes introduced by a specific commit onto the
current branch.

``` bash
git cherry-pick <commit>
```

Example:

``` text
main:     A---B---C
                \
hotfix:          D
```

You can apply D to another branch without merging the entire branch.

Useful for:

-   Production hotfixes
-   Backporting fixes
-   Selectively moving a commit

------------------------------------------------------------------------

## Q28. What problems can cherry-pick cause?

### Answer

Cherry-pick creates a new commit with a different commit ID.

Potential problems:

-   Duplicate logical changes
-   Conflicts
-   Complicated history
-   Future merge confusion

Use it deliberately rather than as a substitute for normal branch
integration.

------------------------------------------------------------------------

# 16. Git Stash

## Q29. What is `git stash`?

### Answer

Stash temporarily stores local modifications so you can work on another
branch or perform another operation.

``` bash
git stash
```

View:

``` bash
git stash list
```

Apply:

``` bash
git stash apply
```

Apply and remove:

``` bash
git stash pop
```

Create a named stash:

``` bash
git stash push -m "WIP login changes"
```

------------------------------------------------------------------------

## Q30. Is `git stash` a permanent backup?

### Answer

No.

Stash is intended for temporary local work.

For important work, committing to a private branch is usually safer.

------------------------------------------------------------------------

# 17. Git Tags

## Q31. What is a Git tag?

### Answer

A tag is a reference used to identify a specific point in history.

Commonly used for releases:

``` bash
git tag v1.0.0
git push origin v1.0.0
```

Annotated tag:

``` bash
git tag -a v1.0.0 -m "Release 1.0.0"
```

------------------------------------------------------------------------

## Q32. Lightweight vs annotated tag?

### Answer

A lightweight tag is essentially a reference to a commit.

An annotated tag is a Git object containing metadata such as:

-   Tagger
-   Message
-   Target object
-   Optional signature

For releases, annotated tags are generally preferable.

------------------------------------------------------------------------

# 18. `.gitignore`

## Q33. What is `.gitignore`?

### Answer

`.gitignore` specifies files Git should normally ignore when they are
untracked.

Typical examples:

``` text
.env
*.log
node_modules/
.terraform/
```

Important:

If a file is already tracked, adding it to `.gitignore` does not remove
it from Git tracking.

Use:

``` bash
git rm --cached <file>
```

then commit the change.

------------------------------------------------------------------------

# 19. Removing Sensitive Data

## Q34. A password was accidentally committed. Is deleting the file enough?

### Answer

No.

Removing the file in a later commit does not remove the secret from Git
history.

The correct response is:

1.  Rotate/revoke the exposed credential immediately.
2.  Assess where it may have been copied.
3.  Remove the secret from repository history using an appropriate
    history-rewriting tool.
4.  Coordinate any forced update of shared repositories.
5.  Search for other copies.
6.  Add secret scanning/prevention controls.

Important principle:

**Treat an exposed credential as compromised even if the commit is later
removed.**

------------------------------------------------------------------------

# 20. Git Log

## Q35. How do you inspect Git history effectively?

### Answer

Useful commands:

``` bash
git log --oneline
git log --graph --oneline --decorate --all
git log --stat
git log -p
```

A very useful troubleshooting command:

``` bash
git log --graph --oneline --decorate --all
```

It visualizes branches and merges.

------------------------------------------------------------------------

# 21. Git Diff

## Q36. Explain `git diff` vs `git diff --staged`.

### Answer

``` bash
git diff
```

Shows changes in the working tree that are not staged.

``` bash
git diff --staged
```

Shows changes currently staged for the next commit.

This distinction is important before committing.

------------------------------------------------------------------------

# 22. Git Bisect

## Q37. What is `git bisect`?

### Answer

`git bisect` performs a binary search through commit history to identify
which commit introduced a bug.

Start:

``` bash
git bisect start
git bisect bad
git bisect good <known-good-commit>
```

Git checks out candidate commits.

You test each candidate and mark:

``` bash
git bisect good
```

or:

``` bash
git bisect bad
```

Eventually Git identifies the problematic commit.

Finish:

``` bash
git bisect reset
```

This is extremely useful when a regression was introduced somewhere
across many commits.

------------------------------------------------------------------------

# 23. Git Blame

## Q38. What is `git blame`?

### Answer

`git blame` identifies the commit and author associated with each line
of a file.

``` bash
git blame <file>
```

Useful for understanding when and where a particular line was
introduced.

It should not be treated as a tool for assigning personal blame; it is
primarily a history investigation tool.

------------------------------------------------------------------------

# 24. Git Clean

## Q39. What does `git clean` do?

### Answer

It removes untracked files from the working tree.

Preview first:

``` bash
git clean -n
```

Then:

``` bash
git clean -f
```

For untracked directories:

``` bash
git clean -fd
```

Be careful because this can permanently remove local untracked work.

------------------------------------------------------------------------

# 25. Detached HEAD

## Q40. What is detached HEAD?

### Answer

Normally HEAD points to a branch:

``` text
HEAD
 |
main
 |
commit
```

In detached HEAD state, HEAD points directly to a commit instead of a
branch.

Example:

``` bash
git checkout <commit>
```

or:

``` bash
git switch --detach <commit>
```

If you make useful commits there, create a branch:

``` bash
git switch -c recovery-branch
```

------------------------------------------------------------------------

# 26. Git Merge vs Rebase Scenario

## Q41. Your feature branch is behind main and has local commits. What would you do?

### Answer

First update remote information:

``` bash
git fetch origin
```

Then, for a private feature branch, I may rebase:

``` bash
git switch feature
git rebase origin/main
```

Resolve conflicts if necessary:

``` bash
git add <file>
git rebase --continue
```

Then push:

``` bash
git push --force-with-lease
```

If the branch is shared and rewriting its history is undesirable, I
would generally merge instead.

------------------------------------------------------------------------

# 27. Pull Request Workflow

## Q42. Describe a good Git workflow for a DevOps team.

### Answer

A common workflow is:

``` text
main
 |
 +---- feature branch
          |
          +---- commits
          |
          v
      Pull Request
          |
          +---- CI
          +---- Tests
          +---- Security Scan
          +---- Code Review
          |
          v
        Merge
          |
          v
         main
```

Typical controls:

-   Protected main branch
-   Pull Requests
-   Required reviews
-   Automated CI checks
-   Security scanning
-   No direct production changes
-   Controlled release tags

------------------------------------------------------------------------

# 28. Branch Protection

## Q43. What is branch protection?

### Answer

Branch protection prevents unsafe changes to important branches.

Typical policies:

-   Require Pull Request
-   Require approvals
-   Require successful CI checks
-   Restrict force pushes
-   Restrict deletion
-   Require conversation resolution
-   Require signed commits where appropriate

The exact controls depend on the Git hosting platform.

------------------------------------------------------------------------

# 29. Git Hooks

## Q44. What are Git hooks?

### Answer

Git hooks are scripts triggered by Git lifecycle events.

Examples:

``` text
pre-commit
commit-msg
pre-push
post-merge
```

They can be used for:

-   Formatting
-   Linting
-   Commit-message validation
-   Security checks
-   Local automation

Important:

Client-side hooks are not a complete security control because users can
bypass or replace them. Critical validation should also run in
CI/server-side controls.

------------------------------------------------------------------------

# 30. Git LFS

## Q45. What is Git LFS?

### Answer

Git Large File Storage stores large files outside normal Git object
storage while Git tracks pointer files.

Useful for:

-   Large binaries
-   Media
-   Machine-learning artifacts
-   Large datasets

It prevents large binary files from unnecessarily inflating normal Git
repository history.

------------------------------------------------------------------------

# 31. Git Performance

## Q46. Repository has become very large. How would you investigate?

### Answer

I would inspect:

``` bash
git count-objects -vH
git rev-list --objects --all
```

Then identify large objects/files.

Potential causes:

-   Large binaries
-   Generated artifacts
-   Build output
-   Accidentally committed archives
-   Historical secrets/files

Depending on the situation, options include:

-   Git LFS
-   Repository history cleanup
-   Better `.gitignore`
-   Preventing generated artifacts from being committed
-   Repository maintenance/GC

History rewriting must be coordinated carefully if the repository is
shared.

------------------------------------------------------------------------

# 32. Git Garbage Collection

## Q47. What is `git gc`?

### Answer

Git garbage collection performs repository maintenance, including
packing objects and pruning objects that are no longer reachable when
safe to do so.

Modern Git automatically performs maintenance in many situations.

You should not use aggressive cleanup casually on a repository where you
may need unreachable objects for recovery.

------------------------------------------------------------------------

# 33. Git Remote Troubleshooting

## Q48. `git push` fails with "non-fast-forward". What does it mean?

### Answer

It usually means the remote branch contains commits that the local
branch does not contain.

First inspect:

``` bash
git fetch origin
git log HEAD..origin/main --oneline
```

Then integrate the remote changes using an appropriate strategy:

``` bash
git pull --rebase
```

or:

``` bash
git pull
```

depending on the team's workflow.

Avoid immediately using:

``` bash
git push --force
```

because it can overwrite remote history.

------------------------------------------------------------------------

# 34. Authentication Failure

## Q49. Git push suddenly asks for credentials or fails authentication. What do you check?

### Answer

First check the remote:

``` bash
git remote -v
```

Then determine whether the repository uses:

-   SSH
-   HTTPS
-   Credential manager
-   Personal access token
-   Enterprise identity/SSO

For SSH:

``` bash
ssh -T git@<git-host>
```

Check:

``` bash
ssh-add -l
```

For HTTPS, verify the configured credential/token and whether the token
has expired or lacks required permissions.

Do not put credentials directly into repository URLs or scripts.

------------------------------------------------------------------------

# 35. Git Reflog Recovery Scenario

## Q50. You rebased the wrong branch and lost your original commits. How do you recover?

### Answer

Use:

``` bash
git reflog
```

Find the HEAD position before the rebase.

Inspect it:

``` bash
git show <old-commit>
```

Create a recovery branch:

``` bash
git switch -c recovery <old-commit>
```

Then compare the recovered history and decide how to restore the
intended branch.

------------------------------------------------------------------------

# 36. Senior L2 Production Scenarios

## Q51. A developer says their commit disappeared. How do you investigate?

### Answer

I would not assume it was deleted.

First inspect:

``` bash
git log --all --oneline --decorate
git reflog
```

Also check:

``` bash
git branch -a
git fsck --no-reflogs
```

if necessary.

Possible causes:

-   Branch was reset
-   Rebase rewrote history
-   Commit exists on another branch
-   Commit was never pushed
-   Branch was deleted
-   Remote branch was force-pushed

If the commit is recoverable, create a branch pointing to it before
doing further cleanup.

------------------------------------------------------------------------

## Q52. Someone force-pushed `main` and removed other developers' commits. What do you do?

### Answer

This is a production-level Git incident.

First:

1.  Stop further destructive pushes.
2.  Identify the previous remote state.
3.  Check local clones/reflogs for the lost commit history.
4.  Recover the correct commit graph.
5.  Coordinate restoration with the team.
6.  Protect the branch against unrestricted force pushes.

A recovery may involve resetting the remote branch to the correct
commit, but that should be coordinated carefully.

The key point is:

**Do not blindly force-push again. Establish the correct desired state
first.**

------------------------------------------------------------------------

## Q53. A merge introduced a production bug. How do you identify which commit caused it?

### Answer

If the approximate range is known, I can use:

``` bash
git bisect start
git bisect bad
git bisect good <known-good-commit>
```

Then run the application's test or validation at each candidate.

Git performs a binary search until it identifies the first bad commit.

For large repositories, this can dramatically reduce investigation time.

------------------------------------------------------------------------

# 37. Git Security

## Q54. How would you secure Git usage in a DevOps environment?

### Answer

I would use:

-   Protected branches
-   Pull Requests
-   Required code reviews
-   CI security checks
-   Secret scanning
-   Dependency scanning
-   Least-privilege repository access
-   MFA/SSO where available
-   Short-lived credentials where supported
-   Signed commits/tags where appropriate
-   `.gitignore` for sensitive/generated files
-   Secure credential storage
-   Audit logging

Never treat `.gitignore` as a secret-management mechanism.

------------------------------------------------------------------------

# 38. Git + CI/CD

## Q55. How does Git integrate with Jenkins or another CI/CD system?

### Answer

A typical flow is:

``` text
Developer
   |
git push
   |
Remote Repository
   |
Webhook
   |
Jenkins
   |
Checkout
   |
Build
   |
Unit Tests
   |
Security Scan
   |
Artifact
   |
Deployment
```

The CI system should identify the exact commit SHA being built.

This provides traceability:

``` text
Commit SHA
   |
Build
   |
Artifact
   |
Deployment
```

------------------------------------------------------------------------

## Q56. Why is commit SHA important in CI/CD?

### Answer

A branch name can move.

For example:

``` text
main -> A
```

later:

``` text
main -> B
```

The commit SHA uniquely identifies the source revision being built.

For production traceability, I want to know:

``` text
Application Version
      |
      v
Artifact
      |
      v
Commit SHA
```

This supports reliable rollback and auditing.

------------------------------------------------------------------------

# 39. Git Interview Command Sheet

## Repository

``` bash
git init
git clone <url>
git status
git remote -v
```

## Branches

``` bash
git branch
git branch -a
git switch main
git switch -c feature/test
git branch -d feature/test
```

## Changes

``` bash
git diff
git diff --staged
git add <file>
git restore <file>
```

## Commits

``` bash
git commit -m "message"
git commit --amend
git log --oneline
git show <commit>
```

## Remote

``` bash
git fetch origin
git pull
git push
git push -u origin <branch>
```

## Merge

``` bash
git merge <branch>
git merge --abort
```

## Rebase

``` bash
git rebase <branch>
git rebase -i HEAD~5
git rebase --continue
git rebase --abort
```

## Recovery

``` bash
git reflog
git reset
git revert
git fsck
```

## Selective Changes

``` bash
git cherry-pick <commit>
```

## Temporary Work

``` bash
git stash
git stash list
git stash pop
```

## Investigation

``` bash
git blame <file>
git bisect start
git log --graph --oneline --decorate --all
```

------------------------------------------------------------------------

# 40. Top 20 Senior L2 Questions

Before an interview, make sure you can answer these without memorizing
blindly:

1.  Git vs GitHub?
2.  Working tree vs staging area vs repository?
3.  What happens internally during `git commit`?
4.  Merge vs rebase?
5.  Fast-forward vs non-fast-forward merge?
6.  How do you resolve merge conflicts?
7.  What happens during an interactive rebase?
8.  Reset vs revert?
9.  Soft vs mixed vs hard reset?
10. How do you recover a lost commit?
11. What is reflog?
12. Fetch vs pull?
13. `--force` vs `--force-with-lease`?
14. What is cherry-pick and when would you use it?
15. What is detached HEAD?
16. How do you remove a committed secret safely?
17. How do you use `git bisect`?
18. How do you troubleshoot a non-fast-forward push?
19. How do you investigate a huge Git repository?
20. How would you design a safe Git workflow for CI/CD?

------------------------------------------------------------------------

# 41. Senior L2 Answer Strategy

When the interviewer gives you a Git problem, structure the response
like this:

``` text
1. Check current repository state
       |
       v
   git status

2. Inspect history
       |
       v
   git log / git reflog

3. Understand branches/remotes
       |
       v
   git branch -vv
   git remote -v
   git fetch

4. Identify the exact problem
       |
       v
   diff / log / show / bisect

5. Apply the least destructive fix
       |
       v
   merge / rebase / revert / cherry-pick

6. Verify
       |
       v
   git status
   git log
   git diff

7. Push safely
       |
       v
   git push
   or
   git push --force-with-lease
```

------------------------------------------------------------------------

# Final Interview Rule

For Senior L2 Git interviews, demonstrate that you understand **history,
references, collaboration and risk**, not just commands.

A strong answer sounds like:

> "First I will inspect the current repository state and determine
> whether the change exists locally or remotely. I will avoid
> destructive commands until I understand the history. I will use
> `git log`, `git reflog`, `git branch -vv` and `git fetch` as
> appropriate. Once I identify the root cause, I will use the least
> destructive recovery method, verify the resulting history, and then
> push using the safest appropriate strategy."

That demonstrates production-oriented Git troubleshooting rather than
simple command memorization.
