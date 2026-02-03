# Changelog Generator Agent

Generate meaningful, user-focused changelogs from git history.

## Capabilities

- Parse git commits using conventional commit format
- Group changes by category (Added, Fixed, Changed, etc.)
- Generate user-friendly descriptions from technical commits
- Create changelogs for different audiences (developers, users)
- Link to issues and PRs

## When to Use

- Before a release
- When updating CHANGELOG.md
- When generating release notes
- When summarizing recent changes

## Workflow

### 1. Gather Commits

Get commits since last release:

```bash
# Find last tag
git describe --tags --abbrev=0

# Get commits since tag
git log v1.0.0..HEAD --pretty=format:"%H|%s|%b---END---"
```

### 2. Parse Commits

Extract structured information:

```markdown
## Parsed Commits

| Hash | Type | Scope | Description | Breaking | Issues |
|------|------|-------|-------------|----------|--------|
| abc123 | feat | auth | Add OAuth login | No | #123 |
| def456 | fix | api | Resolve timeout | No | #456 |
| ghi789 | feat | api | New response format | Yes | #789 |
```

### 3. Categorize Changes

Group by changelog category:

```markdown
### Added
- (auth) Add OAuth login (#123)
- (dashboard) New analytics panel

### Fixed
- (api) Resolve timeout issue (#456)
- (ui) Fix button alignment

### Changed
- (api) New response format (#789) **BREAKING**

### Security
- Update dependencies for CVE-2024-1234
```

### 4. Enhance Descriptions

Transform technical commits to user-friendly language:

```markdown
## Before (Technical)
- feat(auth): implement OAuth2 PKCE flow with refresh token rotation

## After (User-Friendly)
- You can now sign in with Google, GitHub, or Microsoft accounts
```

### 5. Generate Output

Format for the target audience:

**Developer Changelog:**
```markdown
## [1.2.0] - 2024-01-15

### Added
- **auth**: OAuth2 PKCE flow with refresh token rotation (#123)

### Fixed
- **api**: Increased timeout from 30s to 60s (#456)

### Breaking Changes
- **api**: Response format changed to camelCase (#789)
  - Migration: Update client code to use `userId` instead of `user_id`
```

**User Changelog:**
```markdown
## What's New - January 2024

### Sign in with your existing accounts
You can now log in using Google, GitHub, or Microsoft instead of
creating a new password.

### Improved reliability
We fixed an issue that sometimes caused loading to fail on slow
connections.
```

## Output Format

### Standard Changelog Entry

```markdown
## [X.Y.Z] - YYYY-MM-DD

### Added
- [Scope] Description (Issue/PR reference)

### Changed
- [Scope] Description

### Deprecated
- [Scope] Description

### Removed
- [Scope] Description

### Fixed
- [Scope] Description (Issue/PR reference)

### Security
- [Scope] Description (CVE reference if applicable)

### Breaking Changes
- [Scope] Description
  - Migration: How to update
```

### Release Notes

```markdown
# Release Notes: vX.Y.Z

**Release Date**: YYYY-MM-DD

## Highlights

[1-2 sentence summary of most important changes]

## New Features

### Feature Name
[User-friendly description]

## Improvements

- Improvement 1
- Improvement 2

## Bug Fixes

- Fixed issue where [description]

## Breaking Changes

### Change Name
**What changed**: [Description]
**Why**: [Rationale]
**Migration**: [Steps to update]

## Contributors

Thanks to @contributor1, @contributor2 for their contributions!
```

## Configuration

### Commit Type Mapping

```typescript
const typeToCategory = {
  feat: "Added",
  fix: "Fixed",
  perf: "Changed",
  refactor: "Changed",
  docs: null, // Exclude
  style: null, // Exclude
  test: null, // Exclude
  chore: null, // Exclude
  security: "Security",
  deprecate: "Deprecated",
  remove: "Removed",
};
```

### Scope Formatting

```typescript
const scopeLabels = {
  auth: "Authentication",
  api: "API",
  ui: "User Interface",
  db: "Database",
  perf: "Performance",
};
```

## Checklist

When generating changelog:

- [ ] Found correct commit range (last tag to HEAD)
- [ ] Parsed all commits successfully
- [ ] Categorized by type (Added, Fixed, etc.)
- [ ] Identified breaking changes
- [ ] Linked issues and PRs
- [ ] Enhanced descriptions for target audience
- [ ] Included migration guides for breaking changes
- [ ] Formatted according to Keep a Changelog
