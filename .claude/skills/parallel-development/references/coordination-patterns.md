# Coordination Patterns

## The Coordination Spectrum

```
                Less Coordination                    More Coordination
                       │                                    │
   ┌───────────────────┼────────────────────────────────────┼───────────────┐
   │                   │                                    │               │
   │  Shared-Nothing   │    Shared-Interface    Shared-State│  Shared-All   │
   │  (Best for        │    (Balanced)          (Difficult) │  (Avoid)      │
   │   parallel)       │                                    │               │
   └───────────────────┴────────────────────────────────────┴───────────────┘
```

**Goal:** Move as far left as possible on this spectrum.

---

## Pattern 1: Shared-Nothing (Ideal)

Each worktree operates completely independently. No runtime or development-time dependencies.

### Structure

```
Worktree A                    Worktree B
├── src/feature-a/            ├── src/feature-b/
│   ├── api/                  │   ├── api/
│   ├── components/           │   ├── components/
│   ├── services/             │   ├── services/
│   └── tests/                │   └── tests/
└── (complete isolation)      └── (complete isolation)
```

### When It Works

- New features with no shared code
- Completely independent modules
- Separate packages in monorepo
- Different applications

### Coordination Required

- None during development
- Merge timing (but no conflicts)
- Integration testing after merge

---

## Pattern 2: Shared-Interface

Worktrees share interfaces/types but not implementations.

### Structure

```
Shared (Frozen)               Worktree A          Worktree B
├── src/types/                ├── src/auth/       ├── src/billing/
│   ├── user.ts              │   └── (implements │   └── (uses
│   └── api.ts               │       User type)  │       User type)
└── (read-only)
```

### Implementation

```typescript
// Shared types (frozen)
// src/types/user.ts
export interface User {
  id: string;
  email: string;
  role: 'admin' | 'user';
}

// Worktree A implements
// src/auth/createUser.ts
import { User } from '../types/user';
export function createUser(data: Partial<User>): User { ... }

// Worktree B uses
// src/billing/chargeUser.ts
import { User } from '../types/user';
export function chargeUser(user: User, amount: number) { ... }
```

### When It Works

- Features that share data models
- API producers and consumers
- Frontend/backend split on same data

### Coordination Required

- Agree on interfaces before starting
- Type changes require stopping, coordinating, then resuming
- Interface owner (usually main) approves changes

---

## Pattern 3: Shared-State (Difficult)

Worktrees share state at runtime or build time.

### Structure

```
Shared                        Worktree A          Worktree B
├── src/store/               ├── src/feature-a/  ├── src/feature-b/
│   ├── index.ts             │   └── (reads/     │   └── (reads/
│   └── userSlice.ts         │       writes      │       writes
└── (must coordinate)        │       store)      │       store)
```

### Challenges

- Race conditions in development
- State shape changes break both
- Testing is interdependent
- Merge order matters

### When Unavoidable

- Global app state (Redux, Zustand)
- Database schemas
- Shared caches

### Coordination Required

- Heavy coordination
- State changes are blocking
- Consider: Can you split state by domain?

```typescript
// Instead of one shared store:
store = { users: {...}, products: {...}, orders: {...} }

// Split by domain:
authStore = { user: {...}, session: {...} }      // Worktree A owns
billingStore = { subscription: {...}, payments: {...} }  // Worktree B owns
```

---

## Communication Patterns

### Pattern A: Async Documentation

Communication through documentation files.

```markdown
# .parallel-log.md (in main)

## 2024-01-15

### Worktree A (feature-auth)
- Completed: Login flow
- In Progress: Registration
- Blocked: Need User type definition
- Next: Password reset

### Worktree B (feature-billing)
- Completed: Stripe integration
- In Progress: Subscription UI
- Blocked: None
- Next: Invoice generation
```

**Pros:** No real-time coordination needed
**Cons:** May be stale, async delays

### Pattern B: Git-Based Signaling

Use branches and commits to signal state.

```bash
# Worktree A signals completion
git commit -m "READY: User authentication complete"
git push origin feature-auth

# Main coordinator sees signal
git log --oneline --all --grep="READY:"

# Merge when all READY
git merge feature-auth
git merge feature-billing
```

### Pattern C: Marker Files

Use files to indicate state.

```
project/
├── .ready/
│   ├── feature-auth.ready     # Created when auth is ready
│   └── feature-billing.ready  # Created when billing is ready
└── ...

# Coordinator checks:
ls .ready/
# If all expected .ready files exist, proceed with integration
```

---

## Conflict Prevention

### Pre-Work Checklist

Before starting parallel work:

```markdown
## Ownership Declaration

I am working in: ../project-auth (feature-auth branch)

### Files I Own (Exclusive Write Access)
- src/api/auth.js
- src/services/authService.js
- src/components/LoginForm.js
- src/pages/login.js
- tests/auth/*.test.js

### Files I Read (No Write)
- src/types/user.ts (shared)
- src/utils/constants.js (shared)

### Files I Will Create
- src/api/register.js (new file)
- src/components/RegisterForm.js (new file)

### Confirmation
- [ ] No other worktree owns these files
- [ ] Shared files are frozen for this sprint
- [ ] Main knows my file ownership
```

### Runtime Checks

Add to your workflow:

```bash
# Check for potential conflicts before committing
git diff --name-only origin/main | sort > my-changes.txt
# Compare with other worktrees' change lists
```

### Build-Time Checks

In CI/CD:

```yaml
# .github/workflows/conflict-check.yml
on: pull_request

jobs:
  check-conflicts:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Check for conflicting PRs
        run: |
          # Get all open PRs
          gh pr list --json number,headRefName,files
          # Compare file lists for overlaps
          # Fail if conflicts detected
```

---

## Multi-Agent Coordination for Claude

### Terminal Layout

```
┌─────────────────────────────────────────────────────────────────┐
│ Terminal 1: Main (Integration)                                   │
│ cd ~/project && git checkout main                               │
│ > Watching for merge signals...                                 │
├────────────────────┬────────────────────┬───────────────────────┤
│ Terminal 2: Auth   │ Terminal 3: Billing│ Terminal 4: UI        │
│ cd ../project-auth │ cd ../project-bill │ cd ../project-ui      │
│ > claude           │ > claude           │ > claude              │
│ Working on login...│ Working on stripe..│ Working on dashboard..│
└────────────────────┴────────────────────┴───────────────────────┘
```

### Claude Instance Responsibilities

**Main Instance (Coordinator):**
- Monitors progress of other instances
- Handles merge operations
- Resolves cross-cutting concerns
- Updates shared documentation

**Feature Instances (Workers):**
- Focus only on their slice
- Never touch shared code
- Signal completion via commit message
- Request help by creating issues

### Handoff Protocol

```markdown
## Completion Signal

When a feature is ready to merge:

1. Ensure all tests pass: `npm test`
2. Push with signal: `git push origin feature-auth`
3. Create PR with title: "READY: Feature description"
4. Coordinator reviews and merges

## Blocking Signal

When blocked on another worktree:

1. Create issue: "BLOCKED: Need X from feature-billing"
2. Continue on non-blocked work
3. Blocker worktree prioritizes dependency
4. Resolve issue when unblocked
```

---

## Coordination Anti-Patterns

### The Coordination Tax

```
✗ Every change requires checking with other worktrees
✗ Frequent "are you done yet?" messages
✗ Changes blocked waiting for responses
✗ Meeting to discuss every file change

✓ Slice work so coordination is rare
✓ Async communication (docs, commits)
✓ Clear ownership = no asking needed
✓ Merge points are the only sync points
```

### The Hidden Dependency

```
✗ "I assumed you'd add the user endpoint"
✗ Implicit dependencies discovered at merge
✗ Integration breaks because of assumptions

✓ Explicit interface contracts before starting
✓ Mocks/stubs for unbuilt dependencies
✓ Each worktree works independently
```

### The Moving Target

```
✗ Shared interfaces change mid-sprint
✗ "I had to update the User type"
✗ Other worktrees suddenly break

✓ Freeze interfaces during parallel work
✓ Interface changes = stop all worktrees, update, resume
✓ Version shared contracts if changes are essential
```

---

## Decision Tree: How to Coordinate

```
Need to change a file?
│
├─ Is it in my slice? ──────────────────► YES: Make the change
│
├─ Is it frozen/shared? ────────────────► YES: Request change on main first
│                                               Wait for merge to all worktrees
│
├─ Does another worktree own it? ───────► YES: Create issue/request
│                                               They make the change
│                                               Sync after they push
│
└─ Is ownership unclear? ───────────────► STOP: Clarify ownership first
                                                Update slice documentation
                                                Then proceed
```
