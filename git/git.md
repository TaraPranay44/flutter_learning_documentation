# Comprehensive Git Guide for Tech Leads

## Table of Contents
1. [Core Concepts](#core-concepts)
2. [Git Architecture](#git-architecture)
3. [Common Terminology](#common-terminology)
4. [Basic Commands Deep Dive](#basic-commands-deep-dive)
5. [Advanced Commands You Use Daily](#advanced-commands-you-use-daily)
6. [Branch Management](#branch-management)
7. [Collaboration Workflows](#collaboration-workflows)
8. [Troubleshooting & Recovery](#troubleshooting--recovery)
9. [Best Practices](#best-practices)

---

## Core Concepts

### What is Git?
Git is a distributed version control system where every developer has a complete copy of the repository history. Unlike centralized systems, you can work offline and have full access to the entire project history locally.

### The Three States
Every file in Git exists in one of three states:

1. **Modified**: Changed but not staged
2. **Staged**: Marked for inclusion in next commit
3. **Committed**: Safely stored in local database

### The Three Areas

1. **Working Directory**: Your actual files on disk
2. **Staging Area (Index)**: Holds changes for next commit
3. **Repository (.git directory)**: Where Git stores metadata and object database

---

## Git Architecture

### Objects in Git
Git stores data as a series of snapshots, not differences. Four types of objects:

1. **Blob**: File content (data)
2. **Tree**: Directory structure (pointers to blobs and other trees)
3. **Commit**: Snapshot with metadata (author, message, parent commits)
4. **Tag**: Named reference to a specific commit

### How Commits Work
Each commit contains:
- A tree object (snapshot of files)
- Parent commit(s) reference
- Author and committer info
- Commit message
- SHA-1 hash (unique identifier)

Commits form a directed acyclic graph (DAG), where each commit points to its parent(s).

---

## Common Terminology

### HEAD
**What it is**: A pointer to the current branch reference, which points to the last commit made on that branch.

**Think of it as**: "Where you are right now" in your repository's history.

```bash
# HEAD normally points to a branch
HEAD → refs/heads/main → commit abc123

# Detached HEAD: points directly to a commit
HEAD → commit abc123
```

**Common HEAD notations**:
- `HEAD`: Current commit
- `HEAD~1` or `HEAD^`: Parent commit (one step back)
- `HEAD~2`: Grandparent commit (two steps back)
- `HEAD~3`: Great-grandparent (three steps back)

### Origin
**What it is**: The default name Git gives to the remote repository you cloned from.

**Key points**:
- It's just a naming convention (you could name it anything)
- `origin/main` means the `main` branch on the `origin` remote
- Multiple remotes can exist (origin, upstream, etc.)

```bash
# View remotes
git remote -v

# origin is typically your fork or main repo
origin  https://github.com/you/repo.git (fetch)
origin  https://github.com/you/repo.git (push)
```

### Upstream
**What it is**: Refers to the original repository you forked from, or the remote branch your local branch is tracking.

**Two meanings**:

1. **As a remote name**: The original repo (in fork workflows)
```bash
upstream  https://github.com/original/repo.git
```

2. **As a tracking relationship**: The remote branch your local branch tracks
```bash
# Set upstream for current branch
git branch --set-upstream-to=origin/main

# Push and set upstream
git push -u origin feature-branch
```

### Remote vs Local Branches
- **Local branch**: `main`, `feature-branch`
- **Remote-tracking branch**: `origin/main`, `origin/feature-branch`
  - Read-only copies of remote branches
  - Updated when you fetch/pull

### Refs (References)
**What they are**: Human-readable names that point to commits.

Types:
- **Branch refs**: `refs/heads/main`
- **Remote refs**: `refs/remotes/origin/main`
- **Tag refs**: `refs/tags/v1.0.0`

---

## Basic Commands Deep Dive

### git init
Creates a new Git repository by creating a `.git` directory.

```bash
git init
# Creates: .git/objects, .git/refs, .git/HEAD, etc.
```

### git clone
Copies a remote repository to your local machine.

```bash
git clone <url>

# What happens:
# 1. Creates directory
# 2. Initializes .git
# 3. Adds 'origin' remote
# 4. Fetches all data
# 5. Checks out main branch
```

### git add
Moves changes from working directory to staging area.

```bash
git add file.txt          # Stage specific file
git add .                 # Stage all changes in current directory
git add -A                # Stage all changes (entire repo)
git add -p                # Interactively stage chunks
```

**Why staging exists**: Allows you to craft commits precisely, including only related changes.

### git commit
Creates a snapshot of staged changes.

```bash
git commit -m "message"           # Commit with message
git commit -am "message"          # Stage all tracked files and commit
git commit --amend               # Modify last commit
git commit --amend --no-edit     # Amend without changing message
```

### git status
Shows state of working directory and staging area.

```bash
git status
git status -s    # Short format
```

### git log
Shows commit history.

```bash
git log                           # Full log
git log --oneline                 # Condensed view
git log --graph --all             # Visual branch structure
git log -n 5                      # Last 5 commits
git log --author="John"           # By author
git log --since="2 weeks ago"     # By date
git log --follow file.txt         # Follow file through renames
git log -p                        # Show diffs
```

### git diff
Shows differences between commits, branches, files.

```bash
git diff                    # Working directory vs staging
git diff --staged           # Staging vs last commit
git diff HEAD               # Working directory vs last commit
git diff branch1 branch2    # Between branches
git diff commit1 commit2    # Between commits
```

---

## Advanced Commands You Use Daily

### git rebase
**What it does**: Moves or combines commits to a new base commit. Rewrites history by creating new commits.

**Why use it**: 
- Clean, linear history
- Integrate upstream changes without merge commits
- Organize your work before sharing

**How it works**:
```bash
# You're on feature branch
git rebase main

# What happens:
# 1. Git finds common ancestor of feature and main
# 2. Saves your commits (C3, C4) temporarily
# 3. Resets feature to match main
# 4. Replays your commits one by one on top of main
```

**Visual example**:
```
Before:
    A---B---C  main
         \
          D---E  feature

After rebase:
    A---B---C  main
             \
              D'---E'  feature
```

**Important commands**:
```bash
git rebase main                  # Rebase current branch onto main
git rebase -i HEAD~3            # Interactive rebase (last 3 commits)
git rebase --continue           # Continue after resolving conflicts
git rebase --abort              # Cancel rebase
git rebase --skip               # Skip current commit

# Interactive rebase options:
# pick - use commit
# reword - use commit but edit message
# edit - use commit but stop for amending
# squash - combine with previous commit
# fixup - like squash but discard message
# drop - remove commit
```

**When NOT to rebase**: Never rebase commits that have been pushed to shared branches (main, develop) that others work on.

### git reflog
**What it does**: Shows a log of where HEAD has been. Your safety net!

**Why use it**: Recover "lost" commits, undo mistakes, see complete history of your actions.

```bash
git reflog
# Output shows:
# abc123 HEAD@{0}: commit: Add feature
# def456 HEAD@{1}: rebase: checkout main
# ghi789 HEAD@{2}: commit: Fix bug

# Recover "lost" work
git reflog
git reset --hard HEAD@{2}    # Go back to that state

# Or create a branch from old commit
git branch recovery-branch HEAD@{2}
```

**reflog keeps entries for 90 days by default** - it's your time machine.

### git reset
**What it does**: Moves HEAD and current branch to a different commit. Three modes control what happens to staging and working directory.

**The three modes**:

1. **--soft**: Moves HEAD, keeps staging and working directory
   - Use case: Undo commits but keep changes staged
   ```bash
   git reset --soft HEAD~1
   # Undoes last commit, changes stay staged
   ```

2. **--mixed** (default): Moves HEAD, unstages changes, keeps working directory
   - Use case: Undo commits and unstaging, but keep file changes
   ```bash
   git reset HEAD~1
   # or
   git reset --mixed HEAD~1
   # Undoes last commit, changes become unstaged
   ```

3. **--hard**: Moves HEAD, clears staging and working directory
   - Use case: Completely discard commits and changes
   ```bash
   git reset --hard HEAD~1
   # Undoes last commit, DELETES all changes
   # Dangerous! Use reflog to recover if needed
   ```

**Visual comparison**:
```
Initial state:
Working Directory: modified files
Staging Area: some files staged
Repository: commits A-B-C-D (HEAD here)

After git reset --soft HEAD~1:
Working Directory: modified files (unchanged)
Staging Area: some files staged (unchanged)
Repository: commits A-B-C (HEAD here now)

After git reset --mixed HEAD~1:
Working Directory: modified files (unchanged)
Staging Area: empty (cleared)
Repository: commits A-B-C (HEAD here now)

After git reset --hard HEAD~1:
Working Directory: clean (cleared)
Staging Area: empty (cleared)
Repository: commits A-B-C (HEAD here now)
```

**Common uses**:
```bash
# Unstage a file
git reset HEAD file.txt

# Undo last commit, keep changes
git reset --soft HEAD~1

# Discard all local changes
git reset --hard HEAD

# Go back to specific commit
git reset --hard abc123
```

### git cherry-pick
**What it does**: Applies changes from specific commits onto current branch. Creates new commits with same changes.

**Why use it**:
- Apply a bug fix from one branch to another
- Pick specific features without merging entire branch
- Recover specific commits

```bash
# Pick single commit
git cherry-pick abc123

# Pick multiple commits
git cherry-pick abc123 def456

# Pick a range (exclusive start, inclusive end)
git cherry-pick abc123..def456

# Cherry-pick without committing (stage changes only)
git cherry-pick -n abc123

# Continue after resolving conflicts
git cherry-pick --continue

# Abort
git cherry-pick --abort
```

**Example scenario**:
```
You have:
main:     A---B---C
feature:  A---B---D---E---F

You want commit E on main without D or F:
git checkout main
git cherry-pick <E's hash>

Result:
main:     A---B---C---E'
feature:  A---B---D---E---F
```

### git stash
**What it does**: Temporarily saves changes without committing, then cleans working directory.

**Why use it**:
- Switch branches with uncommitted changes
- Pull latest changes without committing
- Quickly test something on clean state

```bash
# Save changes
git stash
git stash save "descriptive message"

# List stashes
git stash list
# Output:
# stash@{0}: On main: descriptive message
# stash@{1}: WIP on feature: abc123 commit message

# Apply stash (keeps stash in list)
git stash apply
git stash apply stash@{1}

# Pop stash (applies and removes from list)
git stash pop

# Apply specific files from stash
git stash show -p stash@{0} | git apply

# Drop stash
git stash drop stash@{0}

# Clear all stashes
git stash clear

# Stash including untracked files
git stash -u

# Stash including ignored files
git stash -a
```

### git merge
**What it does**: Combines branches by creating a merge commit (in most cases).

**Types of merges**:

1. **Fast-forward**: No merge commit needed
```bash
# When target branch hasn't diverged
git merge feature
```

2. **Three-way merge**: Creates merge commit
```bash
# When branches have diverged
git merge feature
# Creates merge commit with two parents
```

3. **Squash merge**: Combines all commits into one
```bash
git merge --squash feature
git commit -m "Merged feature"
```

**Merge vs Rebase**:
- **Merge**: Preserves complete history, creates merge commits
- **Rebase**: Linear history, rewrites commits
- Use merge for shared branches, rebase for local work

---

## Branch Management

### Creating and Switching Branches
```bash
# Create branch
git branch feature-name

# Switch to branch
git checkout feature-name

# Create and switch (shortcut)
git checkout -b feature-name

# Modern Git (2.23+)
git switch feature-name           # Switch
git switch -c feature-name        # Create and switch

# Create from specific commit
git branch feature-name abc123
```

### Viewing Branches
```bash
git branch                  # Local branches
git branch -r               # Remote branches
git branch -a               # All branches
git branch -v               # With last commit
git branch --merged         # Branches merged into current
git branch --no-merged      # Branches not merged
```

### Deleting Branches
```bash
# Delete local branch (safe, prevents if unmerged)
git branch -d feature-name

# Force delete
git branch -D feature-name

# Delete remote branch
git push origin --delete feature-name
```

### Tracking Branches
```bash
# Set upstream for current branch
git branch --set-upstream-to=origin/main

# Push and set upstream
git push -u origin feature-name

# See tracking relationships
git branch -vv
```

---

## Collaboration Workflows

### Fetching Changes
**What it does**: Downloads objects and refs from remote, doesn't modify working directory.

```bash
git fetch                    # Fetch from origin
git fetch origin            # Explicit remote
git fetch --all             # All remotes
git fetch --prune           # Remove deleted remote branches
```

**Fetch vs Pull**:
- **Fetch**: Download only (safe)
- **Pull**: Fetch + merge (can cause conflicts)

### Pulling Changes
```bash
git pull                    # Fetch + merge
git pull --rebase          # Fetch + rebase (cleaner history)
git pull --ff-only         # Only if fast-forward possible
```

### Pushing Changes
```bash
git push                           # Push to upstream
git push origin branch-name        # Push specific branch
git push -u origin branch-name     # Push and set upstream
git push --force                   # Force push (dangerous!)
git push --force-with-lease        # Safer force push
git push --all                     # Push all branches
git push --tags                    # Push tags
```

**When to force push**:
- After rebase on your feature branch
- Never on shared branches
- Use `--force-with-lease` to prevent overwriting others' work

### Working with Forks

**Standard fork workflow**:
```bash
# 1. Clone your fork
git clone https://github.com/you/repo.git
cd repo

# 2. Add upstream remote
git remote add upstream https://github.com/original/repo.git

# 3. Fetch upstream changes
git fetch upstream

# 4. Merge upstream into your branch
git checkout main
git merge upstream/main

# 5. Push to your fork
git push origin main
```

---

## Troubleshooting & Recovery

### Undo Last Commit (Keep Changes)
```bash
git reset --soft HEAD~1
```

### Undo Last Commit (Discard Changes)
```bash
git reset --hard HEAD~1
```

### Recover Deleted Commits
```bash
git reflog
git cherry-pick abc123
# or
git reset --hard HEAD@{2}
```

### Undo Changes to File
```bash
# Unstage file
git restore --staged file.txt

# Discard working directory changes
git restore file.txt

# Old syntax
git checkout -- file.txt
```

### Fix Wrong Commit Message
```bash
git commit --amend -m "Correct message"
```

### Add Forgotten Files to Last Commit
```bash
git add forgotten-file.txt
git commit --amend --no-edit
```

### Resolve Merge Conflicts
```bash
# 1. See conflicted files
git status

# 2. Edit files (remove conflict markers)
# 3. Mark as resolved
git add resolved-file.txt

# 4. Complete merge
git commit

# Or abort
git merge --abort
```

### Clean Untracked Files
```bash
git clean -n          # Dry run (see what would be deleted)
git clean -f          # Delete untracked files
git clean -fd         # Delete files and directories
git clean -fx         # Delete files and ignored files
```

### Find When Bug Was Introduced
```bash
git bisect start
git bisect bad                # Current commit is bad
git bisect good abc123        # Known good commit
# Git checks out middle commit, you test
git bisect good               # If it works
git bisect bad                # If it doesn't
# Repeat until Git finds the commit
git bisect reset              # Return to original state
```

---

## Best Practices

### Commit Messages
Follow conventional format:
```
<type>(<scope>): <subject>

<body>

<footer>
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

Example:
```
feat(auth): add JWT authentication

Implement JWT-based authentication system with
refresh token support and secure cookie storage.

Closes #123
```

### Branch Naming
```
feature/user-authentication
bugfix/login-error
hotfix/security-patch
refactor/payment-system
```

### Workflow Guidelines

1. **Keep commits atomic**: One logical change per commit
2. **Commit often**: Small, frequent commits are better
3. **Pull before push**: Always sync before sharing
4. **Review before commit**: Check `git diff --staged`
5. **Write meaningful messages**: Future you will thank you
6. **Don't commit sensitive data**: Use `.gitignore`
7. **Test before commit**: Ensure code works
8. **Rebase local branches**: Keep history clean
9. **Merge shared branches**: Don't rebase public history
10. **Use branches**: Never commit directly to main

### Git Config Best Practices
```bash
# Set your identity
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Better default branch name
git config --global init.defaultBranch main

# Colorful output
git config --global color.ui auto

# Useful aliases
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.cm commit
git config --global alias.lg "log --graph --oneline --all"

# Pull rebase by default
git config --global pull.rebase true

# Automatically prune on fetch
git config --global fetch.prune true
```

### .gitignore Essentials
```gitignore
# Dependencies
node_modules/
vendor/

# Environment files
.env
.env.local

# IDE
.vscode/
.idea/

# OS
.DS_Store
Thumbs.db

# Build outputs
dist/
build/
*.log

# Sensitive
secrets/
*.key
*.pem
```

---

## Quick Reference

### Daily Commands
```bash
git status                   # Check status
git add .                    # Stage all
git commit -m "message"      # Commit
git pull --rebase           # Update branch
git push                     # Share changes
git log --oneline           # View history
git diff                     # See changes
```

### Emergency Commands
```bash
git reflog                   # Find lost commits
git reset --hard HEAD~1      # Undo last commit
git rebase --abort          # Cancel rebase
git merge --abort           # Cancel merge
git stash                   # Save work quickly
git clean -fd               # Remove untracked files
```

### Interview-Ready Points

1. **Git is distributed**: Every clone is a full repository
2. **Three states**: Modified, staged, committed
3. **HEAD**: Pointer to current branch/commit
4. **Origin**: Default remote repository name
5. **Rebase rewrites history**: Never rebase shared branches
6. **Reflog is your safety net**: Recover "lost" work
7. **Reset modes**: `--soft` (stage), `--mixed` (unstage), `--hard` (discard)
8. **Cherry-pick creates new commits**: Same changes, different hash
9. **Merge preserves history**: Rebase creates linear history
10. **Fetch is safe, pull can conflict**: Always fetch first when unsure

---

## Additional Resources

- Official Git Documentation: https://git-scm.com/doc
- Pro Git Book (free): https://git-scm.com/book
- Git branching visualization: https://learngitbranching.js.org
- Atlassian Git Tutorials: https://www.atlassian.com/git/tutorials

---

**Good luck with your interview! Remember: understanding the "why" behind commands is more valuable than memorizing syntax.**