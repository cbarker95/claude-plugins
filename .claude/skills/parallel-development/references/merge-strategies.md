# Merge Strategies

## Merge Philosophy

The goal of parallel development is to enable independent work, but eventually everything merges to main. Good merge strategy:

- **Minimizes conflict surface** — Small, frequent merges
- **Preserves history** — Clear record of what was merged
- **Maintains stability** — Main is always deployable
- **Enables rollback** — Can revert any merge independently

---

## Merge Workflows

### The Incremental Merge (Recommended)

Merge each feature branch independently as it completes.

```
                    Day 1        Day 2        Day 3
                      │            │            │
feature-auth    ──○───○───○───────┼────────────┼──────
                              │   │            │
                              ▼   │            │
main            ─────────────○────┼────────────┼──────
                                  │            │
feature-billing ─────────○────○───○───○────────┼──────
                                      │        │
                                      ▼        │
main            ─────────────────────○─────────┼──────
                                               │
feature-ui      ───────────────○───────○───○───○
                                               │
                                               ▼
main            ──────────────────────────────○──────
```

**Steps:**
1. Feature-auth finishes → Merge to main
2. Other worktrees rebase on main
3. Feature-billing finishes → Merge to main
4. Remaining worktrees rebase
5. Continue until all merged

**Pros:**
- Small, manageable merges
- Conflicts are localized
- Main stays current

**Cons:**
- Requires rebasing other branches
- More merge commits

### The Sequential Merge

Wait for all features, merge one by one.

```
feature-auth    ──○───○───○───○────────────────────
feature-billing ─────○───○───○───○─────────────────
feature-ui      ────────○───○───○───○──────────────
                                     │  │  │
                                     ▼  ▼  ▼
main            ────────────────────○──○──○────────
```

**Steps:**
1. All features complete independently
2. Merge feature-auth to main
3. Merge feature-billing to main
4. Merge feature-ui to main
5. Resolve conflicts at each step

**Pros:**
- No rebasing during development
- Simpler workflow

**Cons:**
- Conflicts pile up
- Bigger merge operations

### The Octopus Merge (Rare)

Merge all branches simultaneously.

```bash
git merge feature-auth feature-billing feature-ui -m "Merge all features"
```

**When to use:**
- Only if branches have zero overlap
- Typically monorepo with separate packages

**Avoid when:**
- Any possibility of conflicts
- Need to test features individually

---

## Pre-Merge Checklist

Before merging any feature branch:

```markdown
## Pre-Merge Verification

### 1. Branch is Up to Date
- [ ] Fetched latest main: `git fetch origin`
- [ ] Rebased on main: `git rebase origin/main`
- [ ] No conflicts remaining

### 2. Tests Pass
- [ ] All unit tests pass: `npm test`
- [ ] Integration tests pass: `npm run test:integration`
- [ ] Linting passes: `npm run lint`

### 3. Build Succeeds
- [ ] Build completes: `npm run build`
- [ ] No type errors (TypeScript)
- [ ] No runtime errors

### 4. Documentation Updated
- [ ] README updated if needed
- [ ] API docs updated if endpoints changed
- [ ] Migration notes if breaking changes

### 5. Ready for Review
- [ ] Changes summarized in PR
- [ ] No debug code or console.logs
- [ ] No hardcoded test data
```

---

## Merge Commands

### Standard Merge

```bash
# Ensure main is current
git checkout main
git pull origin main

# Merge with merge commit (preserves history)
git merge --no-ff feature-auth -m "Merge feature-auth: User authentication

- Add login/logout flow
- Add registration with email verification
- Add password reset functionality

Resolves: #123, #124"
```

### Squash Merge

Combine all commits into one (cleaner history, less traceability).

```bash
git checkout main
git merge --squash feature-auth
git commit -m "Add user authentication (#123)

Implements login, registration, and password reset."
```

### Rebase and Merge

Replay commits on top of main (linear history).

```bash
# On feature branch
git checkout feature-auth
git rebase origin/main

# On main
git checkout main
git merge feature-auth  # Fast-forward merge
```

---

## Conflict Resolution

### Step 1: Identify Conflicts

```bash
git merge feature-auth
# Auto-merging src/api/users.js
# CONFLICT (content): Merge conflict in src/api/users.js
# Automatic merge failed; fix conflicts and then commit the result.

# List conflicting files
git diff --name-only --diff-filter=U
```

### Step 2: Understand the Conflict

```bash
# View conflict markers
cat src/api/users.js
```

```javascript
function getUser(id) {
<<<<<<< HEAD
  // Main's version
  return db.users.findById(id);
=======
  // Feature's version
  return db.users.findOne({ id });
>>>>>>> feature-auth
}
```

### Step 3: Resolve the Conflict

**Option A: Keep one version**
```bash
git checkout --ours src/api/users.js    # Keep main's version
git checkout --theirs src/api/users.js  # Keep feature's version
```

**Option B: Manual merge**
Edit the file to combine both changes correctly:
```javascript
function getUser(id) {
  // Combined: use findOne with proper syntax
  return db.users.findOne({ _id: id });
}
```

**Option C: Use merge tool**
```bash
git mergetool
# Opens configured merge tool (VS Code, vimdiff, etc.)
```

### Step 4: Complete the Merge

```bash
git add src/api/users.js
git commit -m "Merge feature-auth: Resolve user lookup conflict"
```

---

## Conflict Prevention

### Pre-Merge Sync

Before merging, ensure the feature branch has the latest main:

```bash
# On feature branch
git fetch origin
git rebase origin/main

# If conflicts, resolve them in the feature branch
# This moves conflicts to development, not integration

git push --force-with-lease  # Update remote branch
```

### File Locking (Documentation-Based)

Document who owns what:

```markdown
# .parallel-ownership.md

## Exclusive Ownership

| File/Pattern | Owner | Branch |
|-------------|-------|--------|
| src/api/auth/* | team-auth | feature-auth |
| src/api/billing/* | team-billing | feature-billing |
| src/components/auth/* | team-auth | feature-auth |

## Frozen Files (No Changes)

- package.json
- tsconfig.json
- src/utils/shared.js
```

---

## Post-Merge Tasks

### Notify Other Worktrees

After merging to main:

```bash
# Other worktrees should sync
cd ../project-billing
git fetch origin
git rebase origin/main

# Resolve any conflicts
# Continue development
```

### Run Integration Tests

```bash
cd main-project
git checkout main
npm run test:integration

# If tests fail, investigate or revert
git revert -m 1 HEAD  # Revert the merge if necessary
```

### Clean Up

```bash
# Delete merged branch locally
git branch -d feature-auth

# Delete merged branch remotely
git push origin --delete feature-auth

# Remove worktree
git worktree remove ../project-auth

# Prune stale worktree references
git worktree prune
```

---

## Rollback Strategies

### Revert a Merge

If a merged feature causes issues:

```bash
# Identify the merge commit
git log --oneline --merges
# abc1234 Merge feature-auth: User authentication

# Revert the merge
git revert -m 1 abc1234 -m "Revert feature-auth: Caused production issues"

# -m 1 means "keep main's parent" (undo the merge)
```

### Re-Merge After Fix

If you reverted but want to re-merge after fixes:

```bash
# On feature branch, fix the issue
git checkout feature-auth
# ... make fixes ...
git commit -m "Fix: Resolve production issue"

# Revert the revert on main
git checkout main
git revert HEAD  # Undo the revert

# Now merge the fixed branch
git merge feature-auth
```

---

## Merge Commit Messages

### Template

```
Merge [branch-name]: [Brief description]

[Detailed description of what this merge adds]

Changes:
- [Change 1]
- [Change 2]
- [Change 3]

Related Issues: #123, #124
Breaking Changes: [None | Description]
```

### Examples

```
Merge feature-auth: User authentication system

Adds complete user authentication including login, registration,
and password reset flows. Uses JWT tokens for session management.

Changes:
- Add login/logout endpoints
- Add registration with email verification
- Add password reset via email
- Add JWT token middleware

Related Issues: #123, #124, #125
Breaking Changes: None
```

```
Merge feature-billing: Stripe integration

Integrates Stripe for subscription management and one-time payments.
Includes webhook handling for subscription lifecycle events.

Changes:
- Add Stripe SDK integration
- Add subscription CRUD endpoints
- Add webhook handler for Stripe events
- Add billing history page

Related Issues: #200, #201
Breaking Changes: Requires STRIPE_SECRET_KEY environment variable
```

---

## Merge Timing Guidelines

### When to Merge

- Feature is complete and tested
- All pre-merge checks pass
- No known conflicts with pending merges
- Team is aware (if coordination needed)

### When NOT to Merge

- End of day Friday (no one around if issues)
- During production incidents
- When other merges are in progress
- When major refactoring is pending in another branch

### Merge Order

If multiple branches ready simultaneously:

1. **Smallest diff first** — Less risk, easier to revert
2. **Most urgent first** — Business priority
3. **Dependencies first** — If B depends on A, merge A first
4. **Infrastructure before features** — Foundation first

---

## Quick Reference

| Task | Command |
|------|---------|
| Merge with commit | `git merge --no-ff branch -m "message"` |
| Squash merge | `git merge --squash branch && git commit` |
| Rebase before merge | `git rebase origin/main` |
| Abort merge | `git merge --abort` |
| List conflicts | `git diff --name-only --diff-filter=U` |
| Keep main's version | `git checkout --ours file` |
| Keep feature's version | `git checkout --theirs file` |
| Revert merge | `git revert -m 1 <merge-commit>` |
| Delete branch | `git branch -d branch` |
| Delete remote branch | `git push origin --delete branch` |
