# Changelog Generation

## The Principle

**Changelogs should tell users what changed and why it matters, not just what commits were made.**

A good changelog is a curated, human-readable summary of changes. It's not a git log dump.

---

## Changelog vs Git Log

```
❌ Git Log (Not a changelog)
- abc123 fix typo
- def456 WIP
- ghi789 address review comments
- jkl012 actually fix the bug
- mno345 Merge branch 'main'

✅ Changelog (User-focused)
### Fixed
- Resolved issue where login would timeout after 30 seconds
```

---

## Changelog Format

### Keep a Changelog Format

The standard format from [keepachangelog.com](https://keepachangelog.com):

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- New feature description

### Changed
- Changed behavior description

### Deprecated
- Feature being phased out

### Removed
- Removed feature

### Fixed
- Bug fix description

### Security
- Security fix description

## [1.1.0] - 2024-01-15

### Added
- User authentication via OAuth

### Fixed
- Login timeout issue (#123)

## [1.0.0] - 2024-01-01

### Added
- Initial release
```

### Categories

| Category | What Goes Here |
|----------|----------------|
| Added | New features |
| Changed | Changes to existing features |
| Deprecated | Features being phased out |
| Removed | Removed features |
| Fixed | Bug fixes |
| Security | Security fixes |

---

## Conventional Commits

Use conventional commits to enable automated changelog generation:

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### Types

| Type | Description | Changelog Category |
|------|-------------|-------------------|
| feat | New feature | Added |
| fix | Bug fix | Fixed |
| docs | Documentation | (often excluded) |
| style | Formatting | (often excluded) |
| refactor | Code restructure | Changed |
| perf | Performance | Changed |
| test | Tests | (often excluded) |
| chore | Maintenance | (often excluded) |
| breaking | Breaking change | Changed/Removed |
| security | Security fix | Security |
| deprecate | Deprecation | Deprecated |

### Examples

```bash
# Feature
git commit -m "feat(auth): add OAuth login support"

# Bug fix with issue reference
git commit -m "fix(login): resolve timeout issue

Increases timeout from 30s to 60s.

Fixes #123"

# Breaking change
git commit -m "feat(api)!: change response format

BREAKING CHANGE: API responses now use camelCase keys"

# With scope
git commit -m "fix(dashboard): correct chart rendering on Safari"
```

---

## Automated Generation

### Using conventional-changelog

```bash
# Install
npm install -g conventional-changelog-cli

# Generate changelog
conventional-changelog -p angular -i CHANGELOG.md -s

# First time (generate full history)
conventional-changelog -p angular -i CHANGELOG.md -s -r 0
```

### Using standard-version

```bash
# Install
npm install -D standard-version

# Add to package.json
{
  "scripts": {
    "release": "standard-version",
    "release:minor": "standard-version --release-as minor",
    "release:major": "standard-version --release-as major"
  }
}

# Run
npm run release
```

### Custom Script

```typescript
// scripts/generate-changelog.ts
import { execSync } from "child_process";

interface Commit {
  hash: string;
  type: string;
  scope?: string;
  description: string;
  breaking: boolean;
  issues: string[];
}

function getCommitsSinceTag(tag: string): Commit[] {
  const log = execSync(
    `git log ${tag}..HEAD --pretty=format:"%H|%s|%b---END---"`
  ).toString();

  return log.split("---END---").filter(Boolean).map(parseCommit);
}

function parseCommit(raw: string): Commit {
  const [hash, subject, body] = raw.split("|");

  // Parse conventional commit format
  const match = subject.match(/^(\w+)(?:\(([^)]+)\))?(!)?:\s*(.+)$/);

  if (!match) {
    return { hash, type: "other", description: subject, breaking: false, issues: [] };
  }

  const [, type, scope, bang, description] = match;
  const breaking = !!bang || body?.includes("BREAKING CHANGE");
  const issues = [...(body?.matchAll(/#(\d+)/g) || [])].map(m => m[1]);

  return { hash, type, scope, description, breaking, issues };
}

function generateChangelog(commits: Commit[]): string {
  const categories = {
    Added: commits.filter(c => c.type === "feat"),
    Fixed: commits.filter(c => c.type === "fix"),
    Changed: commits.filter(c => ["refactor", "perf"].includes(c.type)),
    Security: commits.filter(c => c.type === "security"),
    Deprecated: commits.filter(c => c.type === "deprecate"),
  };

  let changelog = "";

  for (const [category, items] of Object.entries(categories)) {
    if (items.length === 0) continue;

    changelog += `### ${category}\n\n`;
    for (const commit of items) {
      const scope = commit.scope ? `**${commit.scope}**: ` : "";
      const issues = commit.issues.length
        ? ` (${commit.issues.map(i => `#${i}`).join(", ")})`
        : "";
      changelog += `- ${scope}${commit.description}${issues}\n`;
    }
    changelog += "\n";
  }

  // Breaking changes section
  const breaking = commits.filter(c => c.breaking);
  if (breaking.length > 0) {
    changelog += `### Breaking Changes\n\n`;
    for (const commit of breaking) {
      changelog += `- ${commit.description}\n`;
    }
    changelog += "\n";
  }

  return changelog;
}

// Usage
const lastTag = execSync("git describe --tags --abbrev=0").toString().trim();
const commits = getCommitsSinceTag(lastTag);
const changelog = generateChangelog(commits);
console.log(changelog);
```

---

## AI-Assisted Generation

Use AI to improve changelog entries:

```typescript
async function enhanceChangelog(commits: Commit[]): Promise<string> {
  const prompt = `
Given these git commits, generate a user-friendly changelog:

${commits.map(c => `- ${c.type}: ${c.description}`).join("\n")}

Requirements:
1. Group by category (Added, Fixed, Changed, etc.)
2. Write in user-friendly language (not developer jargon)
3. Focus on impact, not implementation
4. Keep each entry to one line
5. Include issue references if available

Format as markdown.
`;

  const response = await ai.generate(prompt);
  return response;
}
```

### Before/After AI Enhancement

```
Before (raw commits):
- fix(auth): handle edge case in token refresh logic
- feat(api): add pagination to list endpoints

After (AI enhanced):
### Added
- API list endpoints now support pagination for better performance with large datasets

### Fixed
- Resolved issue where users were unexpectedly logged out during long sessions
```

---

## Changelog Best Practices

### DO

```markdown
✅ Write for your audience (users, not developers)
✅ Group related changes together
✅ Link to issues/PRs for context
✅ Highlight breaking changes prominently
✅ Use consistent formatting
✅ Include migration steps for breaking changes
✅ Date each release
```

### DON'T

```markdown
❌ Include every single commit
❌ Use technical jargon without explanation
❌ Leave entries vague ("fixed bugs")
❌ Skip breaking change notices
❌ Mix user-facing and internal changes
❌ Forget to update before release
```

### Entry Quality

```
❌ Bad: "Fixed bug"
❌ Bad: "Updated dependencies"
❌ Bad: "Refactored code"

✅ Good: "Fixed issue where search results wouldn't load on slow connections"
✅ Good: "Updated authentication library to address security vulnerability CVE-2024-1234"
✅ Good: "Improved page load time by 40% through code optimization"
```

---

## Changelog Workflow

### Per-Commit

```bash
# Write good commit messages
git commit -m "feat(search): add fuzzy matching for better results

Users can now find items even with typos in their search.

Closes #456"
```

### Per-PR

```markdown
<!-- PR template -->
## Changelog Entry

<!-- How should this appear in the changelog? -->
- Added fuzzy search matching for more forgiving search queries
```

### Pre-Release

```bash
# Generate changelog
npm run changelog

# Review and edit
code CHANGELOG.md

# Commit
git add CHANGELOG.md
git commit -m "docs: update changelog for v1.2.0"
```

---

## Multiple Changelogs

For different audiences:

```
CHANGELOG.md          # Developer-focused (full detail)
CHANGELOG-USER.md     # User-focused (simplified)
SECURITY.md           # Security advisories only
```

### User-Focused Changelog

```markdown
# What's New

## January 2024

### Easier Search
Finding what you need is now simpler! Just start typing and we'll
show you matches even if you make a typo.

### Faster Loading
Pages now load up to 40% faster, especially on slower connections.

### Bug Fixes
- Fixed an issue where you might get logged out unexpectedly
- Resolved a display problem on Safari browsers
```

---

## Summary

Effective changelog generation:

1. **Use conventional commits** — Enable automation
2. **Write for users** — Not developers
3. **Categorize changes** — Added, Fixed, Changed, etc.
4. **Highlight breaking changes** — Make them unmissable
5. **Automate generation** — But review and edit
6. **AI-enhance** — Improve clarity and user-focus
7. **Multiple formats** — For different audiences
