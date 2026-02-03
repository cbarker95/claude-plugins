---
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Task, AskUserQuestion, TodoWrite, TaskOutput
description: Setup, sync, or merge git worktrees for parallel development
---

# Parallel Development Command

Manages git worktrees for multi-agent parallel development. Can spawn background agents to work in parallel without requiring the user to open multiple terminals.

## Usage

```
/parallel [subcommand]
```

**Subcommands:**
- `spawn` - **Create worktrees AND spawn background agents** (fully automated)
- `setup` - Create new worktrees (manual terminal launch)
- `sync` - Sync current worktree with main branch
- `merge` - Merge completed feature to main
- `status` - Show status of all worktrees and background agents
- `clean` - Remove a worktree and its branch

---

## Subcommand: spawn (Automated Parallel Execution)

Creates worktrees, generates task prompts, and opens Terminal windows where Claude instances work in parallel. The user approves permissions in each terminal as agents work.

**Why terminal tabs instead of background agents:** Background Task agents cannot get user permission prompts — Bash and Write tools are auto-denied when prompts are unavailable. Interactive terminals solve this.

### Workflow

#### 1. Gather Context

Read the plan file if it exists:

```bash
# Look for active plan
cat ~/.claude/plans/*.md | head -100
```

Or ask user what work to parallelize:

```
AskUserQuestion: "What work should I parallelize?

Options:
- Follow the current plan (if plan file exists)
- Split current PRD features into parallel work
- Custom tasks (specify)"
```

#### 2. Create Worktrees

```bash
PROJECT=$(basename $(pwd))

# Create worktrees for each parallel task
git worktree add ../${PROJECT}-task-1 -b feature-task-1
git worktree add ../${PROJECT}-task-2 -b feature-task-2
```

#### 3. Generate Task Prompts

Write a `.claude-task.md` file in each worktree with the full task specification:

```bash
# Write task prompt for worktree 1
cat > ../${PROJECT}-task-1/.claude-task.md << 'EOF'
# Task: [Task Name]

You are working in worktree: /path/to/${PROJECT}-task-1
Branch: feature-task-1

## Your Task
[Specific task description from plan]

## Steps
1. [Read relevant files]
2. [Create/modify files in scope]
3. [Commit with COMPLETE: prefix]

## Constraints
- Only modify files in your assigned scope
- Do NOT modify: [frozen files]
- Commit when done with message starting with "COMPLETE:"

Begin working now.
EOF
```

Repeat for each worktree with its specific task.

#### 4. Open Terminal Windows

Use `osascript` to open a Terminal window for each worktree, launching Claude with the task prompt:

```bash
# Open Terminal for worktree 1
osascript -e '
tell application "Terminal"
    activate
    do script "cd /path/to/${PROJECT}-task-1 && claude \"$(cat .claude-task.md)\""
end tell'

# Open Terminal for worktree 2
osascript -e '
tell application "Terminal"
    activate
    do script "cd /path/to/${PROJECT}-task-2 && claude \"$(cat .claude-task.md)\""
end tell'
```

#### 5. Track Progress

Create `.parallel-agents.md` in main worktree:

```markdown
# Parallel Development Tracking

**Started**: [date]
**Task**: [description]

## Active Worktrees

| Worktree | Branch | Task | Status |
|----------|--------|------|--------|
| ../${PROJECT}-task-1 | feature-task-1 | [description] | Running |
| ../${PROJECT}-task-2 | feature-task-2 | [description] | Running |

## Completion Check

Run from main worktree:
git log --oneline -1 ../${PROJECT}-task-1  # Look for COMPLETE:
git log --oneline -1 ../${PROJECT}-task-2  # Look for COMPLETE:

## When All Complete

Run: /parallel merge-all
```

#### 6. Monitor Completion

Check each worktree for COMPLETE: commits:

```bash
# Check all worktrees for completion
for worktree in $(git worktree list --porcelain | grep "^worktree" | awk '{print $2}'); do
  echo "$worktree: $(cd $worktree && git log -1 --oneline)"
done
```

### Task Prompt Templates (.claude-task.md)

#### Skill Building Task
```markdown
# Task: Build [skill-name] Skill

You are working in worktree: [worktree path]
Branch: [branch-name]

## Your Task
Create the following files:
1. .claude/skills/[skill-name]/SKILL.md
2. .claude/skills/[skill-name]/references/[ref1].md
3. .claude/skills/[skill-name]/references/[ref2].md
4. .claude/agents/[agent1].md
5. .claude/commands/[command1].md

## Reference Patterns
Read existing skills to follow patterns:
- .claude/skills/product-strategy/SKILL.md (for structure)
- .claude/skills/agent-native-architecture/SKILL.md (for depth)

## Constraints
- Only create files in .claude/ directory
- Do NOT modify existing files
- Commit when done with: "COMPLETE: Add [skill-name] skill"

Begin working now.
```

#### Feature Development Task
```markdown
# Task: Implement [feature-name]

You are working in worktree: [worktree path]
Branch: [branch-name]

## Your Task
[Feature description from PRD]

## Your Scope (only modify these)
- src/features/[feature]/
- src/components/[related]/
- tests/[feature]/

## Do NOT Modify (frozen/shared)
- package.json
- src/lib/

## Constraints
- Commit when done with: "COMPLETE: Implement [feature]"

Begin working now.
```

### Example: Building 2 Doc Guides in Parallel

```
User: /parallel spawn --tasks="skills-guide,commands-guide"

Orchestrator Actions:
1. git worktree add ../toolkit-docs-skills -b feature-docs-skills
2. git worktree add ../toolkit-docs-commands -b feature-docs-commands
3. Write .claude-task.md in each worktree with specific task
4. osascript opens Terminal window 1: cd ../toolkit-docs-skills && claude "$(cat .claude-task.md)"
5. osascript opens Terminal window 2: cd ../toolkit-docs-commands && claude "$(cat .claude-task.md)"
6. Output: "Opened 2 terminals. Approve permissions as agents work."
7. When done: /parallel merge-all
```

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

Shows status of all worktrees AND any background agents.

### Worktree Status

```bash
# List all worktrees
git worktree list

# For each worktree, show:
# - Branch
# - Last commit
# - Status (clean/dirty)
# - Ahead/behind main
```

### Background Agent Status

If `.parallel-agents.md` exists, check each agent:

```typescript
// Read agent tracking file
const agents = parseAgentFile(".parallel-agents.md");

for (const agent of agents) {
  // Check agent status (non-blocking)
  const output = await TaskOutput({
    task_id: agent.id,
    block: false,
    timeout: 1000
  });

  // Check for COMPLETE: commit in worktree
  const lastCommit = exec(`cd ${agent.worktree} && git log -1 --oneline`);
  const isComplete = lastCommit.includes("COMPLETE:");
}
```

### Output Format

```markdown
## Parallel Development Status

### Worktrees

| Worktree | Branch | Status | Behind Main | Last Activity |
|----------|--------|--------|-------------|---------------|
| /path/to/main | main | Clean | - | 2 hours ago |
| /path/to/feature-1 | feature-1 | 3 uncommitted | 2 commits | 30 min ago |
| /path/to/feature-2 | feature-2 | Clean | 0 commits | 1 hour ago |

### Background Agents

| Agent | Worktree | Task | Status | Progress |
|-------|----------|------|--------|----------|
| agent-1 | feature-1 | Build skill | ✅ Complete | COMPLETE: commit found |
| agent-2 | feature-2 | Build skill | 🔄 Running | 3 commits made |

### Actions

- All complete? Run `/parallel merge-all`
- Check agent output: `Read /tmp/agent-[id].out`
- Stop an agent: Use TaskStop tool
```

---

## Subcommand: merge-all

Merges all completed feature branches to main.

### Workflow

1. **Verify all agents complete:**
```typescript
const agents = parseAgentFile(".parallel-agents.md");
const incomplete = agents.filter(a => !isComplete(a));

if (incomplete.length > 0) {
  console.log("Waiting for agents:", incomplete.map(a => a.id));
  return;
}
```

2. **Merge each branch:**
```bash
git checkout main
git pull origin main

# Merge each feature branch
git merge feature-1 --no-edit
git merge feature-2 --no-edit

# Validate
npm test || npm run build
```

3. **Clean up:**
```bash
# Remove worktrees
git worktree remove ../project-feature-1
git worktree remove ../project-feature-2

# Delete tracking file
rm .parallel-agents.md

# Push
git push origin main
```

4. **Report:**
```markdown
## Parallel Merge Complete!

### Merged Branches
- feature-1: [commit hash] - [message]
- feature-2: [commit hash] - [message]

### Validation
- Tests: ✅ Passing
- Build: ✅ Success

### Cleanup
- Worktrees removed: 2
- Branches merged: 2

### Final Commit
[merge commit hash]
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
