---
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Task, AskUserQuestion
description: Setup, sync, or merge git worktrees for parallel development
---

# Parallel Development Command

Manages git worktrees for multi-agent parallel development. Enables multiple Claude instances to work simultaneously on different features.

## Usage

```
/parallel [subcommand]
```

**Subcommands:**
- `setup` - Create new worktrees for parallel development
- `sync` - Sync current worktree with main branch
- `merge` - Merge completed feature to main
- `status` - Show status of all worktrees
- `clean` - Remove a worktree and its branch

---

## Subcommand: setup

Creates parallel development environment with multiple worktrees.

### Workflow

#### 1. Analyze Current State

```bash
# Check we're on main and up to date
git fetch origin
git checkout main
git pull origin main
git status
```

#### 2. Determine Slicing

If the user hasn't specified slices, invoke the vertical-slicer:

```
subagent_type: feature-dev:code-explorer
prompt: "Analyze this codebase to suggest vertical slices for parallel development:

1. Identify major feature areas by directory structure
2. Find shared code that should be frozen
3. Suggest 2-4 independent work streams

Return: Recommended worktree configuration with file ownership."
```

Or ask the user directly:

```
AskUserQuestion: "How would you like to split the work?

Options:
- 2 worktrees (lower overhead)
- 3 worktrees (more parallelism)
- Custom split
- Analyze codebase first"
```

#### 3. Create Worktrees

Based on the configuration:

```bash
# Get project name from current directory
PROJECT=$(basename $(pwd))

# Create worktrees for each slice
git worktree add ../${PROJECT}-feature-1 -b feature-1
git worktree add ../${PROJECT}-feature-2 -b feature-2
# ... more as needed

# Verify
git worktree list
```

#### 4. Generate Documentation

Create `.parallel-dev.md` in main worktree:

```markdown
# Parallel Development Configuration

**Created:** [Today's Date]
**Main Worktree:** [Current Directory]
**Parallel Worktrees:** [Count]

## Active Worktrees

| Worktree | Branch | Focus | Owner |
|----------|--------|-------|-------|
| ../${PROJECT}-feature-1 | feature-1 | [Description] | Available |
| ../${PROJECT}-feature-2 | feature-2 | [Description] | Available |

## File Ownership

### Worktree 1: feature-1
- [List of owned files/directories]

### Worktree 2: feature-2
- [List of owned files/directories]

## Frozen Files (Do Not Modify)
- package.json
- [Other shared files]

## Workflow

1. **Start work:** `cd [worktree-dir] && claude`
2. **Daily sync:** `/parallel sync`
3. **Ready to merge:** Commit with "READY:" prefix
4. **Merge:** `/parallel merge [branch]` from main worktree
```

#### 5. Provide Instructions

Output terminal commands for the user:

```markdown
## Worktrees Created!

To start parallel Claude instances:

**Terminal 1 (Feature 1):**
cd ../${PROJECT}-feature-1 && claude

**Terminal 2 (Feature 2):**
cd ../${PROJECT}-feature-2 && claude

Each instance works on its assigned slice. Sync daily with:
/parallel sync

When ready to merge, return to main and run:
/parallel merge feature-1
```

---

## Subcommand: sync

Synchronizes current worktree with latest main.

### Workflow

```bash
# Save current work
git stash --include-untracked

# Fetch latest
git fetch origin

# Rebase on main
git rebase origin/main

# Handle conflicts if any
# If conflicts: pause and help user resolve

# Run tests
npm test || npm run test || yarn test

# Restore work
git stash pop
```

### Conflict Handling

If rebase conflicts occur:

```
AskUserQuestion: "Rebase conflict detected in:
- [List of conflicting files]

Options:
- Help resolve conflicts (walk through each file)
- Abort rebase (return to previous state)
- Skip problematic commit (rarely appropriate)"
```

---

## Subcommand: merge

Merges a completed feature branch to main.

### Pre-Merge Checklist

1. **Verify in main worktree:**
```bash
git worktree list
# Confirm we're in the main worktree
```

2. **Check feature is ready:**
```bash
# Ensure feature branch has latest main
cd [feature-worktree]
git fetch origin
git rebase origin/main
npm test
```

3. **Ask for confirmation:**
```
AskUserQuestion: "Ready to merge [branch] to main?

Pre-merge verification:
- [ ] Tests passing
- [ ] Rebased on latest main
- [ ] Code reviewed

Options:
- Proceed with merge
- Run tests first
- View changes (git diff)
- Cancel"
```

### Execute Merge

```bash
# Switch to main
git checkout main
git pull origin main

# Merge with descriptive message
git merge --no-ff [branch] -m "Merge [branch]: [Description]

Changes:
- [Change 1]
- [Change 2]

Co-Authored-By: Claude <noreply@anthropic.com>"

# Run integration tests
npm test

# Push
git push origin main
```

### Post-Merge

```markdown
## Merge Complete!

**Merged:** [branch] → main
**Commit:** [hash]

### Next Steps:

1. **Other worktrees should sync:**
   cd ../[other-worktree] && /parallel sync

2. **Clean up merged worktree (optional):**
   /parallel clean [branch]

3. **Continue development:**
   Pick up next task from your slice
```

---

## Subcommand: status

Shows status of all worktrees.

```bash
# List all worktrees
git worktree list

# For each worktree, show:
# - Branch
# - Last commit
# - Status (clean/dirty)
# - Ahead/behind main
```

Output format:

```markdown
## Worktree Status

| Worktree | Branch | Status | Behind Main | Last Activity |
|----------|--------|--------|-------------|---------------|
| /path/to/main | main | Clean | - | 2 hours ago |
| /path/to/feature-1 | feature-1 | 3 uncommitted | 2 commits | 30 min ago |
| /path/to/feature-2 | feature-2 | Clean | 0 commits | 1 hour ago |
```

---

## Subcommand: clean

Removes a worktree and optionally its branch.

### Workflow

1. **Verify worktree exists and is safe to remove:**
```bash
git worktree list
# Check if worktree has uncommitted changes
cd [worktree-path] && git status
```

2. **Confirm with user:**
```
AskUserQuestion: "Remove worktree [path]?

Branch: [branch-name]
Status: [Clean/Has uncommitted changes]
Merged to main: [Yes/No]

Options:
- Remove worktree only (keep branch)
- Remove worktree and branch
- Cancel"
```

3. **Execute removal:**
```bash
# Remove worktree
git worktree remove [path]

# Optionally delete branch
git branch -d [branch]  # Safe delete (fails if not merged)
# or
git branch -D [branch]  # Force delete

# Prune stale references
git worktree prune
```

---

## Error Handling

### "Branch already checked out"

```
The branch [branch] is already checked out in another worktree.

Use a different branch name, or remove the existing worktree first:
  git worktree remove [existing-worktree]
```

### "Worktree has uncommitted changes"

```
Cannot remove worktree with uncommitted changes.

Options:
1. Commit changes: git add . && git commit -m "WIP"
2. Stash changes: git stash
3. Force remove (loses changes): git worktree remove --force [path]
```

### "Merge conflicts"

```
Merge conflict detected. The feature branch has diverged from main.

Recommendation: Resolve conflicts in the feature branch, not main.

1. cd [feature-worktree]
2. git fetch origin
3. git rebase origin/main
4. Resolve conflicts
5. git add . && git rebase --continue
6. Return here and merge again
```

---

## Integration

- Uses `worktree-coordinator` agent for complex operations
- Uses `vertical-slicer` agent to suggest slicing
- References `parallel-development` skill
- Works with multiple Claude instances
