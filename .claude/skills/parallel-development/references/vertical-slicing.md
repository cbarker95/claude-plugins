# Vertical Slicing

## The Problem

When multiple developers (or Claude instances) work in parallel, file conflicts are the enemy:

```
Developer A edits:        Developer B edits:
├── src/api/users.js      ├── src/api/users.js     ← CONFLICT!
├── src/utils/auth.js     ├── src/utils/auth.js    ← CONFLICT!
└── src/pages/profile.js  └── src/pages/settings.js ✓
```

**Horizontal slicing** (by layer) guarantees conflicts because both developers touch the same files at different layers.

**Vertical slicing** (by feature) minimizes conflicts because each developer owns their entire feature stack.

---

## Vertical vs Horizontal

### Horizontal Slicing (Anti-Pattern)

```
Worktree A: "Backend Team"       Worktree B: "Frontend Team"
├── src/api/*                    ├── src/components/*
├── src/services/*               ├── src/pages/*
└── src/utils/*                  └── src/styles/*
```

**Problems:**
- Both need to coordinate on API contracts
- Changes to utils affect both
- Testing requires both to merge first
- Blame game when things break

### Vertical Slicing (Recommended)

```
Worktree A: "User Auth"          Worktree B: "Billing"
├── src/api/auth/*               ├── src/api/billing/*
├── src/services/auth/*          ├── src/services/billing/*
├── src/components/auth/*        ├── src/components/billing/*
├── src/pages/login.js           ├── src/pages/billing.js
└── tests/auth/*                 └── tests/billing/*
```

**Benefits:**
- Each worktree is self-contained
- Can merge independently
- Can test independently
- Clear ownership

---

## How to Slice

### Step 1: Identify Feature Domains

Look for natural boundaries in your codebase:

```
E-commerce Example:
├── User Management (auth, profile, settings)
├── Product Catalog (listing, search, details)
├── Shopping Cart (add, remove, checkout)
├── Order Processing (payment, fulfillment)
└── Admin Dashboard (reports, management)
```

### Step 2: Map Files to Domains

```
User Management:
├── src/api/auth.js
├── src/api/users.js
├── src/services/authService.js
├── src/components/LoginForm.js
├── src/components/UserProfile.js
├── src/pages/login.js
├── src/pages/profile.js
└── tests/auth/*.test.js

Product Catalog:
├── src/api/products.js
├── src/services/productService.js
├── src/components/ProductCard.js
├── src/components/ProductList.js
├── src/pages/products.js
├── src/pages/product/[id].js
└── tests/products/*.test.js
```

### Step 3: Identify Shared Code

Some code is used across domains and should be **frozen** or **isolated**:

```
Shared (Freeze During Parallel Work):
├── src/utils/helpers.js        # Generic utilities
├── src/utils/constants.js      # App-wide constants
├── src/components/Button.js    # Shared UI components
├── src/components/Layout.js    # Layout components
├── src/styles/globals.css      # Global styles
└── package.json                # Dependencies
```

### Step 4: Handle Shared Code

**Option A: Freeze It**
```
Rule: No worktree modifies shared files during parallel phase.
If changes needed, coordinate and do them on main first.
```

**Option B: Split It**
```
Before:
├── src/utils/helpers.js  # 500 lines, used everywhere

After:
├── src/utils/auth-helpers.js     # Used by auth worktree
├── src/utils/billing-helpers.js  # Used by billing worktree
├── src/utils/shared-helpers.js   # Truly shared (frozen)
```

**Option C: Interface It**
```
Define interfaces/contracts that both worktrees implement:
├── src/types/user.ts      # Shared types (frozen)
├── src/api/auth.ts        # Implements user types
└── src/api/billing.ts     # Uses user types
```

---

## Slicing Strategies

### By Feature Area

Best for: Product development, new features

```
Worktree 1: Dashboard Feature
├── src/features/dashboard/*
├── src/pages/dashboard/*
└── tests/dashboard/*

Worktree 2: Reports Feature
├── src/features/reports/*
├── src/pages/reports/*
└── tests/reports/*
```

### By User Type

Best for: Multi-tenant apps, role-based features

```
Worktree 1: Admin Features
├── src/admin/*
├── src/pages/admin/*
└── tests/admin/*

Worktree 2: User Features
├── src/user/*
├── src/pages/user/*
└── tests/user/*
```

### By Integration

Best for: Adding external services

```
Worktree 1: Stripe Integration
├── src/integrations/stripe/*
├── src/api/payments/*
└── tests/payments/*

Worktree 2: SendGrid Integration
├── src/integrations/sendgrid/*
├── src/api/notifications/*
└── tests/notifications/*
```

### By Module (Monorepo)

Best for: Monorepos with packages

```
Worktree 1: Package A
├── packages/package-a/*
└── apps/app-a/*

Worktree 2: Package B
├── packages/package-b/*
└── apps/app-b/*
```

---

## Conflict Risk Assessment

### High Risk (Avoid Parallel Changes)

| Pattern | Risk | Solution |
|---------|------|----------|
| Both edit same file | Guaranteed conflict | Assign file to one worktree |
| Both import same module | Merge conflict on imports | Split module or freeze |
| Both modify package.json | Dependency conflicts | Coordinate, freeze during work |
| Both edit config files | Settings conflicts | Freeze configs |

### Medium Risk (Coordinate)

| Pattern | Risk | Solution |
|---------|------|----------|
| Adding to same directory | Potential file naming conflicts | Communicate naming conventions |
| Different files, shared tests | Test file conflicts | Split test files by feature |
| Different pages, shared layout | Layout import conflicts | Freeze layout during work |

### Low Risk (Parallel Safe)

| Pattern | Why Safe |
|---------|----------|
| Different directories entirely | No overlap |
| Different file extensions | No overlap |
| Adding new files (no edits) | No existing code to conflict |

---

## Documentation Template

Create this file in your main worktree:

```markdown
# Parallel Development Slice Assignment

## Active Slices

### Worktree 1: feature-auth (../project-auth)
**Owner:** Claude Instance 1
**Focus:** User authentication and authorization

**Owned Files:**
- src/api/auth/*
- src/services/auth/*
- src/components/auth/*
- src/pages/login.js
- src/pages/register.js
- tests/auth/*

**May Create:**
- New files in owned directories only

**Must Not Touch:**
- Shared utilities
- Package.json
- Config files

---

### Worktree 2: feature-billing (../project-billing)
**Owner:** Claude Instance 2
**Focus:** Billing and subscription management

**Owned Files:**
- src/api/billing/*
- src/services/billing/*
- src/components/billing/*
- src/pages/billing/*
- tests/billing/*

---

## Frozen Files (No Changes)

These files require coordination:
- package.json
- tsconfig.json
- src/utils/shared/*
- src/components/ui/* (shared components)

## Coordination Points

| File/Area | Owner | How to Request Changes |
|-----------|-------|------------------------|
| package.json | Main | Add to shared deps list, merge to main first |
| Shared types | Main | Define interface, implement in worktree |
| API contracts | Main | Agree on contract before implementing |
```

---

## Validation Checklist

Before starting parallel work:

- [ ] Each worktree has exclusive ownership of its files
- [ ] Shared code is identified and frozen
- [ ] No file appears in multiple worktree slices
- [ ] Each slice can be tested independently
- [ ] Each slice can be merged independently
- [ ] Coordination points are documented
- [ ] Team knows where NOT to make changes

---

## Example: Toolkit Parallel Development

From the plan:

```
Phase 1 (Worktree 1)         Phase 2 (Worktree 2)         Phase 3 (Worktree 3)
─────────────────────       ─────────────────────       ─────────────────────
.claude/skills/              .claude/skills/              .claude/commands/
  mcp-tool-design/             agent-testing/               release.md
.claude/agents/              .claude/agents/                roadmap.md
  mcp-tool-designer.md         parity-test-gen.md        .claude/skills/
  action-parity-*.md           capability-test-*.md         deployment-ops/
```

**Why this works:**
- Each worktree creates new directories (no conflicts)
- Each worktree creates new files (no edits to existing)
- Shared files (existing agents, commands) are frozen
- Can merge in any order without conflicts
