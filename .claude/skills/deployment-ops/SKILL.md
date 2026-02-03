# Deployment Ops Skill

Ship software with confidence through automated releases, changelogs, and CI/CD.

## Purpose

This skill covers the final stages of the product development lifecycle:

- **Release management** — Version control, release planning, feature flags
- **Changelog generation** — Automated, meaningful changelogs from git history
- **CI/CD patterns** — Continuous integration and deployment best practices
- **Shipping workflows** — From code complete to production

## Core Principles

### Ship Incrementally

```
❌ Big Bang Release
   Months of work → One massive deploy → Hope it works

✅ Incremental Shipping
   Small changes → Frequent deploys → Fast feedback → Quick fixes
```

### Automate the Boring Parts

- Changelog generation from commits
- Version bumping
- Release notes compilation
- Deployment triggers

### Keep Humans in the Loop for Decisions

- Release timing
- Feature flag toggles
- Rollback decisions
- Customer communication

## When to Use This Skill

- Planning a release
- Generating changelogs
- Setting up CI/CD pipelines
- Deciding on versioning strategy
- Managing feature flags
- Coordinating deployments

## Key Concepts

### Release Management

Plan and execute releases with confidence. See [release-management.md](references/release-management.md).

### Changelog Generation

Generate meaningful changelogs from git history. See [changelog-generation.md](references/changelog-generation.md).

### CI/CD Patterns

Set up effective continuous integration and deployment. See [ci-cd-patterns.md](references/ci-cd-patterns.md).

## Release Workflow

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Develop   │────▶│   Prepare   │────▶│   Release   │
│             │     │   Release   │     │             │
└─────────────┘     └─────────────┘     └─────────────┘
       │                   │                   │
       ▼                   ▼                   ▼
  Feature work      Version bump         Deploy
  Code review       Changelog            Announce
  Merge to main     Release notes        Monitor
```

### Phase 1: Develop
- Feature branches merged to main
- All tests passing
- Code reviewed

### Phase 2: Prepare Release
- Decide version (major/minor/patch)
- Generate changelog
- Create release notes
- Tag release

### Phase 3: Release
- Deploy to production
- Announce to users
- Monitor for issues
- Be ready to rollback

## Agents

- **changelog-generator** — Generates changelogs from git history
- **release-planner** — Plans releases with versioning and notes

## Commands

- `/release` — Plan and execute a release
- `/roadmap` — Generate product roadmap from issues/PRD
- `/competitive` — Run competitive analysis

## Quick Reference

### Semantic Versioning

```
MAJOR.MINOR.PATCH

MAJOR: Breaking changes
MINOR: New features (backwards compatible)
PATCH: Bug fixes (backwards compatible)

Examples:
1.0.0 → 1.0.1 (bug fix)
1.0.1 → 1.1.0 (new feature)
1.1.0 → 2.0.0 (breaking change)
```

### Conventional Commits

```
feat: add user authentication
fix: resolve login timeout issue
docs: update API documentation
chore: upgrade dependencies
refactor: simplify auth logic
test: add unit tests for auth
perf: optimize database queries
breaking: remove deprecated API endpoints
```

### Release Checklist

```markdown
## Pre-Release
- [ ] All tests passing
- [ ] Changelog updated
- [ ] Version bumped
- [ ] Release notes written
- [ ] Stakeholders notified

## Release
- [ ] Tag created
- [ ] Build successful
- [ ] Deployed to staging
- [ ] Smoke tests passed
- [ ] Deployed to production

## Post-Release
- [ ] Monitoring dashboards checked
- [ ] Announcement published
- [ ] Documentation updated
- [ ] Retrospective scheduled (if major)
```

## References

- [release-management.md](references/release-management.md) — Version control and release planning
- [changelog-generation.md](references/changelog-generation.md) — Automated changelog patterns
- [ci-cd-patterns.md](references/ci-cd-patterns.md) — CI/CD best practices
