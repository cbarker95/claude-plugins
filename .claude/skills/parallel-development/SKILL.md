---
name: parallel-development
description: Git worktrees, multi-agent coordination, vertical slicing. Use when setting up parallel development workflows, splitting work across Claude instances, or coordinating merges from multiple workstreams.
---

<why_now>
## Why Parallel Development Matters

Complex projects benefit from multiple agents working simultaneously on different features. Git worktrees enable this by creating separate working directories that share the same repository, allowing:

- **Parallel execution** — Multiple Claude instances work on different branches simultaneously
- **Clean isolation** — Each worktree has its own index and working tree
- **Shared history** — All worktrees share the same .git directory and commits
- **Conflict minimization** — Vertical slicing keeps worktrees independent

The key insight: Claude instances can work in parallel just like human developers, but they need the right coordination patterns to avoid stepping on each other.
</why_now>

<core_principles>
## Core Principles

### 1. Vertical Slicing

Split work by feature slice, not layer:

```
✗ Horizontal (conflicts guaranteed):
  Worktree A: All backend changes
  Worktree B: All frontend changes

✓ Vertical (minimal conflicts):
  Worktree A: User auth (frontend + backend + tests)
  Worktree B: Billing (frontend + backend + tests)
```

**Test:** Can both worktrees be merged independently without code conflicts?

### 2. Shared-Nothing Architecture

Each worktree should touch different files:

```
Phase 1 Files          Phase 2 Files          Phase 3 Files
─────────────         ─────────────         ─────────────
.claude/skills/       .claude/skills/       .claude/commands/
  mcp-tool-design/      agent-testing/        release.md
.claude/agents/       .claude/agents/         roadmap.md
  mcp-*.md              parity-*.md         .claude/skills/
                                              deployment-ops/
```

**Test:** Do file paths overlap between worktrees?

### 3. Main as Integration Point

Main branch is the integration target, never the work surface:

```
feature-auth ──┬──> main (integration) <──┬── feature-billing
               │                          │
               └─────── merge ────────────┘
```

**Test:** Is anyone coding directly on main?

### 4. Frequent, Small Merges

Merge to main often to avoid divergence:

```
✗ Big bang merge (risky):
  Week 1: Work → Week 2: Work → Week 3: Work → Week 4: Merge chaos

✓ Incremental merge (safe):
  Day 1: Work → Merge → Day 2: Work → Merge → Day 3: Work → Merge
```

**Test:** How long since the last merge to main?

### 5. Coordination Over Communication

Structure the work so coordination happens through the system, not conversation:

```
✗ "Hey, I'm about to edit utils.js, don't touch it"
✓ utils.js only exists in one worktree's slice
```

**Test:** Does parallel work require active coordination?
</core_principles>

<automated_spawning>
## Automated Agent Spawning

The most powerful pattern: spawn background Task agents that work in parallel without requiring the user to open multiple terminals.

### How It Works

```
┌──────────────────────────────────────────────────────────────────┐
│                      Single Claude Instance                       │
├──────────────────────────────────────────────────────────────────┤
│  1. Create worktrees for each task                                │
│  2. Spawn background Task agents with prompts                     │
│  3. Each agent works in its worktree                             │
│  4. Monitor progress via TaskOutput                              │
│  5. Merge all when complete                                       │
└──────────────────────────────────────────────────────────────────┘
         │                    │                    │
         ▼                    ▼                    ▼
    ┌─────────┐          ┌─────────┐          ┌─────────┐
    │ Agent 1 │          │ Agent 2 │          │ Agent 3 │
    │ (bg)    │          │ (bg)    │          │ (bg)    │
    │ Task A  │          │ Task B  │          │ Task C  │
    └─────────┘          └─────────┘          └─────────┘
         │                    │                    │
         └──────── All commit "COMPLETE:" ────────┘
                              │
                              ▼
                       ┌─────────────┐
                       │ Merge All   │
                       │ Clean Up    │
                       └─────────────┘
```

### Usage

```
/parallel spawn --tasks="task-1,task-2,task-3"
```

This will:
1. Create 3 worktrees
2. Spawn 3 background agents with specific prompts
3. Return immediately with agent IDs
4. Agents work in background
5. Use `/parallel status` to check progress
6. Use `/parallel merge-all` when all complete

### Agent Prompt Template

Each spawned agent receives:

```
You are working in worktree: [path]
Branch: [branch-name]

YOUR TASK:
[Specific task from plan or user input]

CONSTRAINTS:
- Only modify files in your assigned scope
- Commit frequently with clear messages
- When done, commit with message starting with "COMPLETE:"

CONTEXT:
[Plan file contents or other context]

Begin working now.
```

### Completion Detection

Agents signal completion by:
1. Making a commit with message starting with "COMPLETE:"
2. The spawn controller checks for this via git log

### Why This Scales

- **No terminal juggling** — User stays in one terminal
- **True parallelism** — All agents work simultaneously
- **Automatic coordination** — Prompts define boundaries
- **Easy monitoring** — Single status command shows all progress
- **Clean merge** — All branches merge in sequence when done
</automated_spawning>

<intake>
## What parallel development task do you need?

1. **Spawn parallel agents** — Create worktrees AND spawn background agents automatically (recommended)
2. **Setup worktrees only** — Create parallel worktrees (manual terminal launch)
3. **Slice work** — Analyze a project and suggest vertical slicing for parallel development
4. **Check status** — View status of all worktrees and running agents
5. **Sync worktree** — Fetch latest changes, rebase, and run tests
6. **Merge all** — Merge all completed feature branches to main
7. **Coordinate work** — Get guidance on multi-agent coordination patterns

**Wait for response before proceeding.**
</intake>

<routing>
| Response | Action |
|----------|--------|
| 1, "spawn", "auto", "background" | Run `/parallel spawn` — Creates worktrees and spawns background Task agents |
| 2, "setup", "create", "worktree" | Read [git-worktree-patterns.md](./references/git-worktree-patterns.md), run `/parallel setup` |
| 3, "slice", "split", "divide" | Read [vertical-slicing.md](./references/vertical-slicing.md), run `/slice` |
| 4, "status", "check", "progress" | Run `/parallel status` — Shows worktrees and background agent progress |
| 5, "sync", "rebase", "update" | Read [merge-strategies.md](./references/merge-strategies.md), run `/parallel sync` |
| 6, "merge", "integrate", "complete" | Run `/parallel merge-all` — Merges all completed branches |
| 7, "coordinate", "multi-agent", "parallel" | Read [coordination-patterns.md](./references/coordination-patterns.md) |

**After reading references, apply patterns to the user's specific context.**
</routing>

<parallel_agents>
## Agent Orchestration

### Setup Phase (Sequential)
```
┌─────────────────────────┐
│ vertical-slicer         │
│ → Analyzes codebase     │
│ → Suggests work splits  │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│ worktree-coordinator    │
│ → Creates worktrees     │
│ → Sets up branches      │
└─────────────────────────┘
```

### Work Phase (Parallel)
```
┌─────────────────────────┐  ┌─────────────────────────┐  ┌─────────────────────────┐
│ Worktree 1              │  │ Worktree 2              │  │ Worktree 3              │
│ → Claude Instance A     │  │ → Claude Instance B     │  │ → Claude Instance C     │
│ → Feature slice 1       │  │ → Feature slice 2       │  │ → Feature slice 3       │
└───────────┬─────────────┘  └───────────┬─────────────┘  └───────────┬─────────────┘
            │                            │                            │
            └────────────┬───────────────┴───────────────┬────────────┘
                         │                               │
                         ▼                               ▼
            ┌─────────────────────────┐    ┌─────────────────────────┐
            │ Daily sync              │    │ Incremental merge       │
            │ /parallel sync          │    │ /parallel merge         │
            └─────────────────────────┘    └─────────────────────────┘
```

### Merge Phase (Sequential)
```
┌─────────────────────────┐
│ worktree-coordinator    │
│ → Merge feature → main  │
│ → Run integration tests │
│ → Clean up worktree     │
└─────────────────────────┘
```
</parallel_agents>

<setup_workflow>
## Worktree Setup Workflow

### Step 1: Analyze Project

Launch vertical-slicer agent:
```
subagent_type: feature-dev:code-explorer
prompt: "Analyze the codebase structure to identify:
1. Major feature areas (directories, modules)
2. Shared code/utilities that shouldn't be touched in parallel
3. Integration points (APIs, shared state, config)
4. Current file organization patterns

Return: Recommended vertical slices for parallel development"
```

### Step 2: Plan Worktrees

Based on analysis, plan N worktrees (typically 2-4):

```markdown
## Proposed Worktrees

| Worktree | Branch | Focus Area | Files |
|----------|--------|------------|-------|
| ../project-auth | feature-auth | Authentication | src/auth/*, api/auth/* |
| ../project-billing | feature-billing | Billing | src/billing/*, api/billing/* |
| ../project-ui | feature-ui | UI polish | src/components/*, styles/* |
```

### Step 3: User Confirmation

Use AskUserQuestion:
```
"Here's the proposed parallel development setup:

Worktree 1: ../project-auth (feature-auth)
- User authentication and authorization
- Files: src/auth/*, api/auth/*

Worktree 2: ../project-billing (feature-billing)
- Billing and subscriptions
- Files: src/billing/*, api/billing/*

This creates 2 parallel workstreams with minimal file overlap.

Options:
- Proceed with this setup
- Adjust the slicing
- Add another worktree
- Start fresh with different approach"
```

### Step 4: Create Worktrees

Execute setup commands:
```bash
# Ensure main is up to date
git checkout main && git pull

# Create worktrees with dedicated branches
git worktree add ../project-auth -b feature-auth
git worktree add ../project-billing -b feature-billing

# Verify setup
git worktree list
```

### Step 5: Document Setup

Create `.parallel-dev.md` in main:
```markdown
# Parallel Development Setup

## Active Worktrees

| Worktree | Branch | Owner | Status |
|----------|--------|-------|--------|
| ../project-auth | feature-auth | Claude-1 | Active |
| ../project-billing | feature-billing | Claude-2 | Active |

## Coordination Rules

1. Each worktree owns its file slice
2. Merge to main daily
3. Rebase from main after merges
4. Resolve conflicts in feature branch, not main

## Shared Files (DO NOT MODIFY)

- package.json (coordinate version bumps)
- tsconfig.json
- .env.example
```
</setup_workflow>

<sync_workflow>
## Daily Sync Workflow

Run in each worktree to stay current:

### Step 1: Stash Local Changes
```bash
git stash  # If uncommitted changes exist
```

### Step 2: Fetch and Rebase
```bash
git fetch origin
git rebase origin/main
```

### Step 3: Run Tests
```bash
npm test  # or your test command
```

### Step 4: Restore and Continue
```bash
git stash pop  # If stashed earlier
```

### Handling Conflicts

If rebase conflicts occur:
1. Identify conflicting files
2. Check if file should be in this worktree's slice
3. If yes: resolve conflict, continue rebase
4. If no: coordinate with other worktree owner

```bash
# After resolving conflicts
git add <resolved-files>
git rebase --continue
```
</sync_workflow>

<merge_workflow>
## Merge Workflow

When a feature is complete:

### Step 1: Final Sync
```bash
git fetch origin
git rebase origin/main
npm test  # Ensure tests pass
```

### Step 2: Merge to Main
```bash
git checkout main
git merge --no-ff feature-auth -m "Merge feature-auth: User authentication"
```

### Step 3: Notify Other Worktrees
Other worktrees should rebase:
```bash
# In other worktrees
git fetch origin
git rebase origin/main
```

### Step 4: Clean Up (Optional)
```bash
# Remove worktree when done
git worktree remove ../project-auth
git branch -d feature-auth  # Delete local branch
```
</merge_workflow>

<reference_index>
## Reference Files

All references in `references/`:

**Setup & Management:**
- [git-worktree-patterns.md](./references/git-worktree-patterns.md) — Worktree commands, directory structure, best practices

**Work Division:**
- [vertical-slicing.md](./references/vertical-slicing.md) — How to split work for minimal conflicts

**Coordination:**
- [coordination-patterns.md](./references/coordination-patterns.md) — Multi-agent coordination, communication patterns

**Integration:**
- [merge-strategies.md](./references/merge-strategies.md) — Merge workflows, conflict resolution, cleanup
</reference_index>

<commands>
## Available Commands

- `/parallel` — Setup, sync, or merge worktrees
- `/slice` — Analyze codebase and suggest vertical slicing
</commands>

<anti_patterns>
## Anti-Patterns

### Setup Anti-Patterns

**Horizontal slicing** — Splitting by layer instead of feature
```
✗ "Worktree A does all backend, B does all frontend"
✓ "Worktree A does auth (full stack), B does billing (full stack)"
```

**Overlapping slices** — Multiple worktrees touching same files
```
✗ Both worktrees modify src/utils/helpers.js
✓ Shared utilities are frozen or split into separate files
```

**Main as workspace** — Doing development directly on main
```
✗ cd project && claude (working on main)
✓ cd project-feature && claude (working on feature branch)
```

### Coordination Anti-Patterns

**Silent divergence** — Not syncing with main for days
```
✗ Work for a week, then face massive merge conflicts
✓ Sync daily, merge when feature slices are complete
```

**Cross-worktree dependencies** — One worktree waiting on another
```
✗ "I can't start until you finish the user model"
✓ Slice work so each worktree can progress independently
```

**Communication over convention** — Relying on messages instead of structure
```
✗ "Don't touch api/users.js, I'm working on it"
✓ api/users.js is exclusively in one worktree's slice
```

### Merge Anti-Patterns

**Big bang merge** — Saving all merges for the end
```
✗ All 3 worktrees merge at once on Friday
✓ Each worktree merges when its slice is complete
```

**Merge commit avalanche** — Too many merge commits cluttering history
```
✗ 50 merge commits for one feature
✓ Rebase feature branch, then single merge to main
```

**Conflict avoidance** — Skipping sync because conflicts are scary
```
✗ "I'll deal with conflicts later"
✓ Small, frequent syncs = small, easy conflicts
```
</anti_patterns>

<success_criteria>
## Success Criteria

You've done parallel development well when:

### Setup
- [ ] Worktrees are created with clear, non-overlapping slices
- [ ] Each worktree has its own feature branch
- [ ] Documentation exists showing who owns what
- [ ] Shared/frozen files are identified

### Execution
- [ ] Each worktree can work independently without blocking
- [ ] Daily syncs happen with minimal conflicts
- [ ] Merge conflicts are small and localized
- [ ] Progress is visible across all worktrees

### Integration
- [ ] Features merge to main incrementally
- [ ] Main branch always passes tests
- [ ] No big-bang merge at the end
- [ ] Worktrees clean up after completion

### The Ultimate Test

**Can you spin up a new Claude instance in a new worktree and have it start working immediately without coordinating with other instances?**

If yes, you've structured parallel development correctly.
</success_criteria>
