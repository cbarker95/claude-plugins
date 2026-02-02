# Git Worktree Patterns

## What Are Git Worktrees?

Git worktrees allow you to have multiple working directories attached to a single repository. Each worktree:

- Has its own working tree and index
- Can be on a different branch
- Shares the same `.git` directory (commits, objects, refs)

```
project/                    # Main worktree (typically main branch)
├── .git/                   # Shared git directory
├── src/
└── ...

project-feature-a/          # Linked worktree (feature-a branch)
├── .git -> ../project/.git # Symlink to shared .git
├── src/
└── ...

project-feature-b/          # Linked worktree (feature-b branch)
├── .git -> ../project/.git
├── src/
└── ...
```

---

## Essential Commands

### Creating Worktrees

```bash
# Create worktree with new branch
git worktree add ../project-feature -b feature-name

# Create worktree on existing branch
git worktree add ../project-feature existing-branch

# Create worktree with specific commit/tag
git worktree add ../project-hotfix v1.0.0 -b hotfix-branch
```

### Listing Worktrees

```bash
git worktree list
# Output:
# /Users/user/project        abc1234 [main]
# /Users/user/project-auth   def5678 [feature-auth]
# /Users/user/project-ui     ghi9012 [feature-ui]

# Detailed view
git worktree list --porcelain
```

### Removing Worktrees

```bash
# Remove worktree (must have clean working tree)
git worktree remove ../project-feature

# Force remove (discards uncommitted changes)
git worktree remove --force ../project-feature

# Prune stale worktree references
git worktree prune
```

### Moving Worktrees

```bash
# Move worktree to new location
git worktree move ../project-feature ../new-location
```

---

## Directory Structure Patterns

### Pattern 1: Sibling Directories (Recommended)

```
~/projects/
├── my-app/                 # Main worktree
├── my-app-feature-auth/    # Feature worktree
├── my-app-feature-ui/      # Feature worktree
└── my-app-hotfix/          # Hotfix worktree
```

**Pros:**
- Easy to navigate
- Clear visual separation
- IDE-friendly (each is a project)

**Cons:**
- Can clutter projects directory

### Pattern 2: Subdirectory

```
~/projects/my-app/
├── main/                   # Main worktree
├── worktrees/
│   ├── feature-auth/       # Feature worktree
│   ├── feature-ui/         # Feature worktree
│   └── hotfix/             # Hotfix worktree
└── .git/                   # Shared git directory
```

**Pros:**
- Contains everything in one place
- Clear hierarchy

**Cons:**
- Nested paths can be confusing

### Pattern 3: Purpose-Based

```
~/work/
├── dev/my-app/             # Main development
├── review/my-app/          # Code review worktree
├── hotfix/my-app/          # Emergency fixes
└── experiment/my-app/      # Experimental work
```

**Pros:**
- Separates by intent
- Works across multiple projects

---

## Naming Conventions

### Branch Names

```bash
# Feature work
feature-auth
feature-billing
feature-ui-polish

# Bug fixes
fix-login-error
fix-payment-validation

# Parallel development phases
phase-1-mcp-design
phase-2-testing
phase-3-shipping
```

### Worktree Directory Names

```bash
# Match branch name
../my-app-feature-auth
../my-app-feature-billing

# Include project prefix for clarity
../toolkit-auth
../toolkit-billing

# Short and memorable
../auth-work
../billing-work
```

---

## Best Practices

### 1. One Branch Per Worktree

Never checkout the same branch in multiple worktrees.

```bash
# This will fail (as expected)
git worktree add ../project-copy main
# fatal: 'main' is already checked out at '/path/to/project'
```

### 2. Keep Main Clean

Main worktree should stay on `main` branch for integration:

```bash
# Main worktree: integration point
cd project/
git checkout main

# Feature work happens in other worktrees
cd ../project-feature/
git checkout feature-branch
```

### 3. Clean Up After Merging

```bash
# After merging feature-auth to main
git worktree remove ../project-auth
git branch -d feature-auth

# If branch is not fully merged, use -D carefully
git branch -D feature-auth
```

### 4. Use Absolute Paths

```bash
# Reliable (absolute path)
git worktree add /Users/user/projects/my-app-feature -b feature

# Can be confusing (relative path)
git worktree add ../my-app-feature -b feature
```

---

## Common Issues

### Worktree Shows as Prunable

```bash
git worktree list
# Shows: /path/to/worktree  abc1234 [branch] prunable

# Fix: Either remove or repair
git worktree remove /path/to/worktree
# or
git worktree repair
```

### Can't Checkout Branch

```bash
# Error: branch is already checked out
git checkout feature
# fatal: 'feature' is already checked out at '/other/worktree'

# Solution: Work in the worktree that has the branch
# or: Create a new branch for your work
```

### Worktree Has Uncommitted Changes

```bash
# Can't remove worktree with uncommitted changes
git worktree remove ../my-worktree
# error: cannot remove: contains modified files

# Options:
# 1. Commit or stash changes
# 2. Force remove (loses changes)
git worktree remove --force ../my-worktree
```

### Detached HEAD in Worktree

```bash
# Worktree might end up with detached HEAD after operations
cd ../my-worktree
git status
# HEAD detached at abc1234

# Fix: checkout the intended branch
git checkout feature-branch
```

---

## IDE Integration

### VS Code

Each worktree can be opened as a separate project:

```bash
code ../project-feature
```

Or use VS Code's multi-root workspace:

```json
// project.code-workspace
{
  "folders": [
    { "path": "project" },
    { "path": "project-feature-auth" },
    { "path": "project-feature-ui" }
  ]
}
```

### Terminal Setup

Useful aliases for worktree navigation:

```bash
# ~/.bashrc or ~/.zshrc
alias wt="git worktree"
alias wtl="git worktree list"
alias wta="git worktree add"
alias wtr="git worktree remove"

# Quick navigation
function cdw() {
  cd "../$(basename $(pwd))-$1"
}
# Usage: cdw feature  # Goes to ../project-feature
```

---

## Worktrees for Claude Instances

### Setup Pattern

```bash
# Main project (coordinator)
cd ~/projects/my-app
git checkout main

# Claude Instance 1
git worktree add ../my-app-feature-1 -b feature-1
cd ../my-app-feature-1 && claude

# Claude Instance 2 (separate terminal)
git worktree add ../my-app-feature-2 -b feature-2
cd ../my-app-feature-2 && claude

# Claude Instance 3 (separate terminal)
git worktree add ../my-app-feature-3 -b feature-3
cd ../my-app-feature-3 && claude
```

### Sync Pattern

```bash
# Each Claude instance periodically syncs
git fetch origin
git rebase origin/main
```

### Merge Pattern

```bash
# When feature is complete (from main worktree)
cd ~/projects/my-app
git checkout main
git pull
git merge --no-ff feature-1 -m "Merge feature-1: Description"
git push
```

---

## Quick Reference

| Command | Purpose |
|---------|---------|
| `git worktree add <path> -b <branch>` | Create worktree with new branch |
| `git worktree add <path> <branch>` | Create worktree on existing branch |
| `git worktree list` | Show all worktrees |
| `git worktree remove <path>` | Remove worktree |
| `git worktree prune` | Clean up stale references |
| `git worktree repair` | Fix worktree references |
| `git worktree move <src> <dest>` | Move worktree location |
