---
allowed-tools: Read, Write, Glob, Grep, Bash, Task, AskUserQuestion
description: Analyze codebase and suggest vertical slicing for parallel development
---

# Slice Command

Analyzes a codebase and recommends how to split work for parallel development with minimal conflicts.

## Purpose

Before setting up parallel worktrees, you need to know how to divide the work. This command:

1. Analyzes your codebase structure
2. Identifies feature domains and boundaries
3. Finds shared code that should be frozen
4. Recommends optimal worktree configurations
5. Documents file ownership to prevent conflicts

---

## Workflow

### Step 1: Understand Goals

Ask what needs to be parallelized:

```
AskUserQuestion: "What work do you want to parallelize?

Context helps me suggest better slices:
- New features to build
- Refactoring tasks
- Multiple bug fixes
- Full project phases

What are you trying to accomplish?"
```

### Step 2: Analyze Codebase

Launch parallel analysis agents:

**Agent 1: Directory Structure**
```
subagent_type: Explore
prompt: "Map the high-level directory structure:

1. What are the top-level directories in src/?
2. What naming conventions are used?
3. How is code organized (by feature, by layer, by type)?
4. What looks like distinct feature areas?

Return: Directory tree with annotations about purpose."
```

**Agent 2: Feature Detection**
```
subagent_type: Explore
prompt: "Identify distinct feature areas in this codebase:

1. What pages/routes exist?
2. What API endpoints are defined?
3. What major components or modules exist?
4. What tests exist and how are they organized?

Return: List of feature domains with their file locations."
```

**Agent 3: Dependency Analysis**
```
subagent_type: Explore
prompt: "Find shared code and dependencies:

1. What utility files are imported by multiple features?
2. What types/interfaces are shared?
3. What configuration files exist?
4. What UI components are reused?

Return: List of shared files that would cause conflicts if modified in parallel."
```

### Step 3: Build Feature Map

Synthesize analysis into domains:

```markdown
## Feature Domain Map

### Domain: [Feature Name]
**Purpose:** [What this feature does]
**Files:**
- src/api/[feature]/
- src/services/[feature]/
- src/components/[feature]/
- src/pages/[feature]/
- tests/[feature]/

**Dependencies:**
- Uses: src/types/[shared-types].ts
- Uses: src/utils/[shared-utils].ts

**Conflict Risk with Other Domains:** [Low/Medium/High]
```

### Step 4: Identify Shared Code

List files that should be frozen:

```markdown
## Shared Code (Freeze During Parallel Work)

### Configuration Files
- package.json - Dependencies affect all
- tsconfig.json - TypeScript config
- tailwind.config.js - Styling config
- .env.example - Environment template

### Shared Types
- src/types/index.ts
- src/types/user.ts
- src/types/api.ts

### Shared Utilities
- src/utils/helpers.ts
- src/utils/constants.ts
- src/lib/api.ts

### Shared UI Components
- src/components/ui/* - Button, Input, etc.
- src/components/layout/* - Header, Footer, etc.

### Reason for Freezing
These files are imported across multiple features. Parallel changes would cause:
- Merge conflicts
- Type incompatibilities
- Build failures
```

### Step 5: Assess Conflict Risk

For each pair of potential slices:

```markdown
## Conflict Risk Matrix

|             | Slice A | Slice B | Slice C |
|-------------|---------|---------|---------|
| **Slice A** | -       | Low     | Medium  |
| **Slice B** | Low     | -       | Low     |
| **Slice C** | Medium  | Low     | -       |

### Risk Details

**A + C Medium Risk:**
- Both may modify src/components/shared/Modal.tsx
- Recommendation: Assign Modal to one slice or freeze it
```

### Step 6: Propose Configurations

Generate 2-3 options:

```markdown
## Recommended Configurations

### Option 1: Conservative (2 Worktrees)

Lower coordination overhead, larger slices.

**Worktree A: [Name]**
- Branch: feature-[name]
- Focus: [Description]
- Files: [Count] files across [X] directories
- Complexity: [Low/Medium/High]

**Worktree B: [Name]**
- Branch: feature-[name]
- Focus: [Description]
- Files: [Count] files
- Complexity: [Low/Medium/High]

**Trade-offs:**
- ✓ Less coordination needed
- ✓ Simpler merge process
- ✗ Less parallelism
- ✗ Larger individual scope

---

### Option 2: Balanced (3 Worktrees)

Good balance of parallelism and coordination.

[Similar structure for 3 worktrees]

---

### Option 3: Aggressive (4 Worktrees)

Maximum parallelism, more coordination.

[Similar structure for 4 worktrees]
```

### Step 7: Present Recommendation

```
AskUserQuestion: "Based on analysis, here are the slicing options:

**Option 1: 2 Worktrees (Recommended)**
- Slice A: [Feature set] - [file count] files
- Slice B: [Feature set] - [file count] files
- Frozen: [count] shared files

**Option 2: 3 Worktrees (More Parallel)**
- Adds a third stream for [feature]
- Slightly more merge coordination

**Option 3: 4 Worktrees (Maximum)**
- Finest granularity
- Requires careful coordination

Which configuration would you like to proceed with?"
```

### Step 8: Generate Slice Document

Write detailed specification:

```markdown
# Parallel Development Slices

**Project:** [Name]
**Generated:** [Date]
**Configuration:** [Option chosen]

---

## Slice 1: [Feature Name]

**Worktree:** ../[project]-[feature]
**Branch:** feature-[name]

### Owned Files (Exclusive Write)

| Path Pattern | Description |
|--------------|-------------|
| src/api/[feature]/* | API routes |
| src/services/[feature]/* | Business logic |
| src/components/[feature]/* | UI components |
| src/pages/[feature]/* | Page components |
| tests/[feature]/* | Test files |

### Files to Create

These new files should be created in this slice:
- src/api/[feature]/new-endpoint.ts
- src/components/[feature]/NewComponent.tsx

### Read-Only Dependencies

These files can be imported but not modified:
- src/types/user.ts
- src/utils/helpers.ts
- src/components/ui/Button.tsx

---

## Slice 2: [Feature Name]

[Similar structure]

---

## Frozen Files

These files must NOT be modified during parallel work:

| File | Reason | Change Process |
|------|--------|----------------|
| package.json | Shared deps | Coordinate, merge to main first |
| src/types/*.ts | Shared types | Define interface on main |
| src/components/ui/* | Shared UI | Freeze or assign to one slice |

---

## Coordination Protocol

### Before Starting
1. All slices verify their file ownership
2. Frozen files are acknowledged
3. Each slice creates its worktree

### During Work
1. Daily sync: `/parallel sync`
2. New shared needs: Create issue, coordinate
3. Conflicts: Resolve in feature branch

### Merge Order
1. [Slice with no dependencies] - Can merge anytime
2. [Slice depending on 1] - After slice 1 merges
3. [Slice depending on 1 or 2] - After dependencies

---

## Quick Start

After approval, run:
\`\`\`bash
/parallel setup
\`\`\`

This will create the worktrees based on this specification.
```

---

## Output

The command produces:

1. **Console summary** - Quick overview of recommended slicing
2. **Slice specification file** - Detailed documentation at `.parallel-slices.md`
3. **Ready for `/parallel setup`** - Can immediately create worktrees

---

## Integration

- Runs before `/parallel setup`
- Uses `vertical-slicer` agent for analysis
- Output feeds into `worktree-coordinator` agent
- References `parallel-development` skill
