# Release Management

## The Principle

**Ship frequently, ship confidently, ship incrementally.**

Good release management reduces the risk and stress of deployments while increasing the pace of delivery.

---

## Versioning Strategies

### Semantic Versioning (SemVer)

The standard for most software:

```
MAJOR.MINOR.PATCH

1.0.0 - Initial release
1.0.1 - Bug fix
1.1.0 - New feature (backwards compatible)
2.0.0 - Breaking change
```

**When to bump what:**

| Change Type | Version Bump | Example |
|-------------|--------------|---------|
| Bug fix | PATCH | 1.0.0 → 1.0.1 |
| New feature | MINOR | 1.0.1 → 1.1.0 |
| Breaking change | MAJOR | 1.1.0 → 2.0.0 |
| Pre-release | Add suffix | 2.0.0-beta.1 |

### CalVer (Calendar Versioning)

For products with regular release cycles:

```
YYYY.MM.DD or YYYY.MM.PATCH

2024.01.15 - Release on Jan 15, 2024
2024.02.1  - First February release
2024.02.2  - Second February release
```

### Build Numbers

For internal/continuous deployment:

```
build-1234
commit-abc123
```

---

## Release Types

### Major Releases

Significant changes, potentially breaking:

```markdown
## Major Release Checklist

### Planning (2-4 weeks before)
- [ ] Define scope and breaking changes
- [ ] Create migration guide
- [ ] Notify stakeholders of timeline
- [ ] Plan deprecation notices

### Preparation (1 week before)
- [ ] Feature freeze
- [ ] Release candidate created
- [ ] Migration guide reviewed
- [ ] Beta testing complete

### Release Day
- [ ] Final changelog review
- [ ] Deploy to production
- [ ] Publish migration guide
- [ ] Send announcement

### Post-Release (1 week after)
- [ ] Monitor for issues
- [ ] Gather feedback
- [ ] Plan hotfixes if needed
- [ ] Retrospective
```

### Minor Releases

New features, backwards compatible:

```markdown
## Minor Release Checklist

### Preparation
- [ ] Features complete and merged
- [ ] Changelog updated
- [ ] Version bumped
- [ ] Release notes drafted

### Release
- [ ] Tag created
- [ ] CI/CD pipeline runs
- [ ] Deploy to production
- [ ] Brief announcement
```

### Patch Releases

Bug fixes, minimal risk:

```markdown
## Patch Release Checklist

- [ ] Fix merged to main
- [ ] Tests passing
- [ ] Version bumped
- [ ] Changelog updated
- [ ] Deploy
```

---

## Release Branches

### Git Flow

```
main (production)
  │
  ├── develop (integration)
  │     │
  │     ├── feature/new-thing
  │     └── feature/other-thing
  │
  └── release/1.2.0 (release prep)
        │
        └── hotfix/urgent-fix
```

### Trunk-Based Development

```
main (always deployable)
  │
  ├── feature/short-lived-1
  ├── feature/short-lived-2
  │
  └── release/1.2.0 (optional, for stabilization)
```

### GitHub Flow

```
main (production)
  │
  ├── feature/thing-1 → PR → main
  ├── feature/thing-2 → PR → main
  └── feature/thing-3 → PR → main
```

---

## Feature Flags

Decouple deployment from release:

```typescript
// Feature flag implementation
const features = {
  newDashboard: {
    enabled: process.env.FF_NEW_DASHBOARD === "true",
    rolloutPercent: 10, // 10% of users
  },
  betaFeature: {
    enabled: false,
    allowList: ["user-123", "user-456"],
  },
};

function isFeatureEnabled(feature: string, userId?: string): boolean {
  const config = features[feature];
  if (!config) return false;

  // Check allow list first
  if (userId && config.allowList?.includes(userId)) {
    return true;
  }

  // Check rollout percentage
  if (config.rolloutPercent) {
    const hash = hashUserId(userId);
    return hash % 100 < config.rolloutPercent;
  }

  return config.enabled;
}

// Usage
if (isFeatureEnabled("newDashboard", user.id)) {
  return <NewDashboard />;
} else {
  return <OldDashboard />;
}
```

### Feature Flag Lifecycle

```
1. Create flag (disabled)
2. Implement feature behind flag
3. Deploy to production (flag off)
4. Enable for internal testing
5. Gradual rollout (10% → 50% → 100%)
6. Remove flag and old code
```

---

## Release Notes

### Structure

```markdown
# Release Notes: v1.2.0

**Release Date**: January 15, 2024

## Highlights

[1-2 sentences on the most important changes]

## New Features

### Feature Name
[Description of what it does and why it matters]

## Improvements

- Improved X for better Y
- Enhanced Z performance

## Bug Fixes

- Fixed issue where [description]
- Resolved [problem]

## Breaking Changes

- `oldFunction()` has been removed. Use `newFunction()` instead.

## Upgrade Guide

[Steps to upgrade from previous version]

## Known Issues

- [Any known issues with workarounds]
```

### Audience-Specific Notes

**For Developers:**
```markdown
## API Changes

### New Endpoints
- `POST /api/v2/users` - Create user with new schema

### Deprecated
- `GET /api/v1/users` - Use v2 instead

### Breaking
- Response format changed for `/api/users/:id`
```

**For End Users:**
```markdown
## What's New

### Easier Project Setup
You can now create projects in one click! Just...

### Faster Search
Search is now 3x faster...
```

---

## Rollback Planning

### Rollback Criteria

Define when to rollback:

```markdown
## Rollback Triggers

### Automatic Rollback
- Error rate > 5% (baseline: 0.1%)
- P50 latency > 500ms (baseline: 100ms)
- Health checks failing

### Manual Rollback Decision
- Critical bug affecting > 1% of users
- Data integrity issues
- Security vulnerability discovered
```

### Rollback Procedure

```markdown
## Rollback Steps

1. **Decide** - Confirm rollback is needed
2. **Communicate** - Alert team in #releases
3. **Execute** - Run rollback command
   ```bash
   ./scripts/rollback.sh v1.1.0
   ```
4. **Verify** - Check health metrics
5. **Investigate** - Root cause analysis
6. **Plan** - Fix and re-release
```

---

## Release Automation

### Automated Version Bumping

```bash
# Using standard-version or similar
npx standard-version --release-as minor

# Or with npm version
npm version minor -m "Release v%s"
```

### Release Scripts

```bash
#!/bin/bash
# scripts/release.sh

set -e

VERSION=$1

echo "Releasing version $VERSION..."

# Ensure we're on main
git checkout main
git pull origin main

# Run tests
npm test

# Bump version
npm version $VERSION -m "Release v%s"

# Generate changelog
npm run changelog

# Commit and tag
git add CHANGELOG.md
git commit --amend --no-edit
git tag -f "v$VERSION"

# Push
git push origin main --tags

# Create GitHub release
gh release create "v$VERSION" \
  --title "v$VERSION" \
  --notes-file RELEASE_NOTES.md

echo "Released v$VERSION!"
```

---

## Release Communication

### Internal Communication

```markdown
## Release Announcement Template (Internal)

**Subject**: [Release] v1.2.0 going out today

**Team**,

We're releasing v1.2.0 today at 2pm PT.

**Key Changes:**
- New feature X
- Fixed bug Y
- Performance improvement Z

**Rollout Plan:**
- 2pm: Deploy to 10% of users
- 3pm: If healthy, expand to 50%
- 4pm: Full rollout

**Rollback Plan:**
- Automatic if error rate > 5%
- Manual trigger in #releases

**On-call**: @alice (primary), @bob (secondary)

Questions? Ping me or drop in #releases.
```

### External Communication

```markdown
## Release Announcement Template (External)

**Subject**: [Product] v1.2.0 Released - New Features!

Hi everyone,

We're excited to announce v1.2.0 of [Product]!

**What's New:**
- 🎉 [Feature 1] - [One-line description]
- ⚡ [Feature 2] - [One-line description]
- 🐛 Bug fixes and improvements

**Getting Started:**
[Link to docs or upgrade guide]

**Feedback:**
We'd love to hear from you! [Link to feedback channel]

Thanks for using [Product]!
```

---

## Summary

Effective release management:

1. **Version clearly** — Use SemVer or CalVer consistently
2. **Plan releases** — Checklists for each release type
3. **Use feature flags** — Decouple deploy from release
4. **Write good notes** — Clear, audience-appropriate release notes
5. **Plan for rollback** — Know when and how to rollback
6. **Automate** — Script repetitive tasks
7. **Communicate** — Keep stakeholders informed
