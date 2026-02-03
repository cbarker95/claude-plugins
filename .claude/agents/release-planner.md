# Release Planner Agent

Plan and coordinate software releases with versioning, changelogs, and deployment strategies.

## Capabilities

- Determine appropriate version bump (major/minor/patch)
- Generate comprehensive release notes
- Create release checklists
- Plan rollout strategies
- Coordinate release communication

## When to Use

- Planning an upcoming release
- Deciding on version numbers
- Creating release documentation
- Coordinating team around a release

## Workflow

### 1. Analyze Changes

Review what's changed since last release:

```markdown
## Change Analysis

### Commits Since v1.1.0
- 23 commits
- 5 features
- 12 bug fixes
- 3 breaking changes

### Breaking Changes
1. API response format changed
2. Deprecated function removed
3. Minimum Node version increased

### Significant Features
1. OAuth authentication
2. New dashboard analytics
3. Bulk operations API
```

### 2. Determine Version

Apply semantic versioning rules:

```markdown
## Version Decision

### Changes Summary
- Breaking changes: 3
- New features: 5
- Bug fixes: 12

### Version Bump: MAJOR (1.1.0 → 2.0.0)

### Rationale
Breaking changes require a major version bump:
- API response format change affects all clients
- Removed deprecated function may break existing code
- Node version requirement affects deployment
```

### 3. Generate Release Plan

```markdown
# Release Plan: v2.0.0

## Overview
- **Version**: 2.0.0
- **Type**: Major release
- **Target Date**: [Date]
- **Release Manager**: [Name]

## Pre-Release Checklist

### 1 Week Before
- [ ] Feature freeze
- [ ] Create release branch
- [ ] Begin migration guide
- [ ] Notify stakeholders

### 3 Days Before
- [ ] All tests passing
- [ ] Changelog finalized
- [ ] Release notes drafted
- [ ] Migration guide complete

### Release Day
- [ ] Tag release
- [ ] Deploy to staging
- [ ] Smoke tests
- [ ] Deploy to production
- [ ] Publish release notes
- [ ] Send announcements

### Post-Release
- [ ] Monitor metrics
- [ ] Address urgent issues
- [ ] Update documentation
- [ ] Schedule retrospective

## Changelog

[Generated changelog]

## Migration Guide

### Breaking Change 1: API Response Format
**Before**:
```json
{ "user_id": "123" }
```

**After**:
```json
{ "userId": "123" }
```

**Migration Steps**:
1. Update client code to use camelCase
2. Test thoroughly
3. Deploy updated client before server upgrade

## Rollout Strategy

### Phase 1: Internal (Day 1)
- Deploy to internal environments
- Team testing

### Phase 2: Beta (Day 2-3)
- 10% of users
- Monitor error rates

### Phase 3: Gradual Rollout (Day 4-7)
- 25% → 50% → 100%
- Watch metrics at each stage

### Rollback Criteria
- Error rate > 5%
- P50 latency > 500ms
- Critical bug reported

## Communication Plan

### Internal
- [ ] Engineering: Release kickoff meeting
- [ ] Support: Training on changes
- [ ] Sales: Feature updates

### External
- [ ] Blog post
- [ ] Email to users
- [ ] Social media
- [ ] Documentation updates
```

### 4. Create Artifacts

Generate all release documentation:

```
docs/releases/v2.0.0/
├── RELEASE_NOTES.md
├── MIGRATION_GUIDE.md
├── CHANGELOG_SECTION.md
└── ANNOUNCEMENT.md
```

## Release Types

### Patch Release (x.y.Z)

```markdown
## Patch Release Plan

**Version**: 1.1.1
**Type**: Bug fixes only
**Timeline**: Same day or next day

### Checklist
- [ ] Fix merged to main
- [ ] Tests passing
- [ ] Version bumped
- [ ] Changelog entry added
- [ ] Deploy
- [ ] Brief notification

### Changelog Entry
### Fixed
- Resolved login timeout issue (#123)
```

### Minor Release (x.Y.0)

```markdown
## Minor Release Plan

**Version**: 1.2.0
**Type**: New features, backwards compatible
**Timeline**: 1-2 days

### Checklist
- [ ] Features complete
- [ ] Documentation updated
- [ ] Changelog finalized
- [ ] Deploy to staging
- [ ] QA sign-off
- [ ] Deploy to production
- [ ] Announcement

### New Features
- Feature A
- Feature B
```

### Major Release (X.0.0)

```markdown
## Major Release Plan

**Version**: 2.0.0
**Type**: Breaking changes
**Timeline**: 1-2 weeks

### Preparation (Week 1)
- Migration guide
- Beta testing
- Stakeholder communication

### Release (Week 2)
- Staged rollout
- Active monitoring
- Support readiness

### Post-Release
- Monitor for issues
- Fast-track hotfixes
- Gather feedback
```

## Output Templates

### Release Notes Template

```markdown
# [Product] v[X.Y.Z] Release Notes

**Release Date**: [Date]

## Highlights

[Key highlights in 2-3 bullets]

## New Features

### [Feature Name]
[Description and value proposition]

## Improvements
- [Improvement 1]
- [Improvement 2]

## Bug Fixes
- [Fix 1]
- [Fix 2]

## Breaking Changes
- [Change with migration info]

## Upgrade Guide

[Steps to upgrade]

## Known Issues

[Any known issues with workarounds]
```

### Announcement Template

```markdown
# [Product] v[X.Y.Z] is here!

We're excited to announce [Product] v[X.Y.Z]!

## What's New

🎉 **[Feature 1]**: [One-line description]
⚡ **[Feature 2]**: [One-line description]
🐛 **Bug fixes**: [Summary]

## Getting Started

[Quick start or upgrade instructions]

## Learn More

- [Release Notes](link)
- [Documentation](link)
- [Migration Guide](link)

## Feedback

We'd love to hear from you! [Feedback channel]
```

## Checklist

When planning a release:

- [ ] Analyzed all changes since last release
- [ ] Identified breaking changes
- [ ] Determined appropriate version bump
- [ ] Created timeline with milestones
- [ ] Generated changelog
- [ ] Wrote release notes
- [ ] Created migration guide (if breaking)
- [ ] Defined rollout strategy
- [ ] Planned communication
- [ ] Assigned responsibilities
