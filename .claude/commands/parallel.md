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

Creates worktrees and spawns background Task agents to work in parallel. This is the recommended approach for automated parallel development.

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

#### 3. Spawn Background Agents

For each worktree, spawn a Task agent in the background:

```typescript
// Spawn agent for worktree 1
Task({
  subagent_type: "general-purpose",
  run_in_background: true,
  prompt: `You are working in worktree: /path/to/${PROJECT}-task-1
Branch: feature-task-1

YOUR TASK:
[Specific task description from plan]

CONSTRAINTS:
- Only modify files in your assigned scope
- Commit frequently with clear messages
- When done, commit with message starting with "COMPLETE:"

CONTEXT:
[Relevant context from plan file]

Begin working on your task now.`
})

// Spawn agent for worktree 2
Task({
  subagent_type: "general-purpose",
  run_in_background: true,
  prompt: `You are working in worktree: /path/to/${PROJECT}-task-2
...`
})
```

#### 4. Track Progress

Store agent IDs and output files:

```markdown
# .parallel-agents.md

## Active Parallel Agents

| Agent ID | Worktree | Task | Output File | Status |
|----------|----------|------|-------------|--------|
| [id-1] | task-1 | [description] | /tmp/agent-1.out | Running |
| [id-2] | task-2 | [description] | /tmp/agent-2.out | Running |

## Commands

Check progress: `/parallel status`
Stop an agent: Use TaskStop with agent ID
Merge results: `/parallel merge-all`
```

#### 5. Monitor Loop

Provide status updates and wait for completion:

```typescript
// Check agent outputs periodically
for (const agent of activeAgents) {
  const output = await TaskOutput({
    task_id: agent.id,
    block: false,  // Non-blocking check
    timeout: 1000
  });

  if (output.status === "completed") {
    // Agent finished, check for COMPLETE: commit
  }
}
```

### Prompt Templates for Spawned Agents

#### Skill Building Agent
```
You are building a skill in: [worktree path]

Create the following files:
1. .claude/skills/[skill-name]/SKILL.md
2. .claude/skills/[skill-name]/references/[ref1].md
3. .claude/skills/[skill-name]/references/[ref2].md
4. .claude/agents/[agent1].md
5. .claude/commands/[command1].md

Follow these patterns:
[Include relevant patterns from existing skills]

When complete, commit with: "COMPLETE: Add [skill-name] skill"
```

#### Feature Development Agent
```
You are implementing a feature in: [worktree path]

Feature: [Feature description from PRD]

Your scope (only modify these):
- src/features/[feature]/
- src/components/[related]/
- tests/[feature]/

Do NOT modify:
- package.json (shared)
- src/lib/ (shared)

When complete, commit with: "COMPLETE: Implement [feature]"
```

### Example: Building 2 Skills in Parallel

```
User: /parallel spawn --tasks="mcp-design,agent-testing"

Agent Actions:
1. git worktree add ../toolkit-mcp-design -b feature-mcp-design
2. git worktree add ../toolkit-agent-testing -b feature-agent-testing
3. Task(background=true, prompt="Build mcp-design skill in /path/to/toolkit-mcp-design...")
4. Task(background=true, prompt="Build agent-testing skill in /path/to/toolkit-agent-testing...")
5. Output: "Spawned 2 agents. Run `/parallel status` to check progress."
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
