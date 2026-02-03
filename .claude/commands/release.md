---
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Task, AskUserQuestion, TodoWrite
description: Plan and execute a software release
---

# Release Command

Plan and execute a software release with versioning, changelog, and deployment.

## Phase 1: Gather Context

Launch these sub-agents **in parallel** using the Task tool:

### Agent 1: Change Analysis
```
subagent_type: Explore
prompt: "Analyze changes since last release:
1. Find the last git tag
2. List all commits since that tag
3. Identify commit types (feat, fix, breaking, etc.)
4. Count changes by category

Return:
- Last version
- Commit count
- Features added
- Bugs fixed
- Breaking changes
- Contributors"
```

### Agent 2: Test Status
```
subagent_type: Bash
prompt: "Check the current test and build status:
1. Run the test suite
2. Run the build
3. Check for any linting errors

Return pass/fail status for each."
```

### Agent 3: Current Version
```
subagent_type: Explore
prompt: "Find the current version:
- Check package.json
- Check version files
- Check recent git tags

Return current version and versioning scheme used."
```

Wait for all agents, then synthesize.

## Phase 2: Version Decision

Based on changes, recommend version bump:

```markdown
## Version Analysis

### Changes Summary
- Breaking changes: [count]
- New features: [count]
- Bug fixes: [count]
- Other: [count]

### Recommendation: [MAJOR/MINOR/PATCH]

Current: v[X.Y.Z]
Proposed: v[A.B.C]

### Rationale
[Why this version bump]
```

Ask user to confirm using AskUserQuestion:

```
"Based on the changes, I recommend a [TYPE] release:

Current: v[X.Y.Z]
Proposed: v[A.B.C]

Key changes:
- [Change 1]
- [Change 2]
- [Breaking change if any]

Which version would you like to release?
- [Recommended version]
- [Alternative]
- Custom version
- Cancel"
```

## Phase 3: Generate Changelog

Using the changelog-generator agent pattern:

1. Parse all commits since last tag
2. Group by category (Added, Fixed, Changed, etc.)
3. Identify breaking changes
4. Link to issues/PRs
5. Generate user-friendly descriptions

Output format:
```markdown
## [X.Y.Z] - YYYY-MM-DD

### Added
- Feature description (#issue)

### Fixed
- Bug fix description (#issue)

### Changed
- Change description

### Breaking Changes
- Breaking change with migration info
```

## Phase 4: Create Release

Execute release steps:

```bash
# 1. Update version
npm version [version] --no-git-tag-version

# 2. Update changelog
# (Prepend generated changelog to CHANGELOG.md)

# 3. Commit changes
git add package.json CHANGELOG.md
git commit -m "chore: release v[version]"

# 4. Create tag
git tag -a v[version] -m "Release v[version]"

# 5. Push
git push origin main --tags
```

## Phase 5: Create GitHub Release

```bash
gh release create v[version] \
  --title "v[version]" \
  --notes-file RELEASE_NOTES.md
```

## Phase 6: Post-Release

Ask about additional actions:

```
"Release v[version] is complete!

What else would you like to do?
- Deploy to production
- Send announcement
- Update documentation
- Nothing, we're done"
```

## Output

### Release Summary

```markdown
# Release Summary: v[X.Y.Z]

## Status: ✅ Complete

### Version
- Previous: v[A.B.C]
- Current: v[X.Y.Z]
- Type: [Major/Minor/Patch]

### Changes Included
- Features: [count]
- Fixes: [count]
- Breaking: [count]

### Artifacts Created
- [x] Version bumped in package.json
- [x] CHANGELOG.md updated
- [x] Git tag created
- [x] GitHub release created

### Links
- [Release](github-release-url)
- [Changelog](changelog-section-url)
- [Commits](compare-url)

### Next Steps
- [ ] Deploy to production
- [ ] Announce to users
- [ ] Update documentation
```

## Quick Mode

For fast patch releases:

```bash
/release --patch --yes
```

Skips confirmations and uses defaults.

## Options

- `--major` — Force major version bump
- `--minor` — Force minor version bump
- `--patch` — Force patch version bump
- `--yes` — Skip confirmations
- `--dry-run` — Show what would happen without executing
- `--no-push` — Don't push to remote

## Related Commands

- `/roadmap` — Plan future releases
- `/dev` — Continue development work
