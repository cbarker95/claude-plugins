# Claude Plugins

A collection of reusable Claude Code customizations: skills, agents, and commands.

## What's Included

### Skills (3)

| Skill | Description |
|-------|-------------|
| **atomic-design-system** | Build and evaluate design systems using Brad Frost's atomic design methodology |
| **agent-native-architecture** | Patterns for building apps where agents are first-class citizens |
| **frontend-design** | Create distinctive, production-grade frontend interfaces with high design quality |

### Agents (13)

| Agent | Description |
|-------|-------------|
| **figma-design-sync** | Synchronize implementations with Figma designs |
| **design-implementation-reviewer** | Verify UI matches Figma specifications |
| **design-iterator** | Iteratively refine designs with screenshot analysis |
| **design-reviewer** | Compare screenshots against Figma designs |
| **accessibility-checker** | WCAG 2.1 AA compliance auditing |
| **atomic-migration-planner** | Plan component migrations to atomic design |
| **atomic-compliance-scorer** | Score codebase against atomic design principles |
| **component-hierarchy-analyzer** | Classify components into atomic levels |
| **token-auditor** | Extract and audit design tokens |
| **framework-docs-researcher** | Gather documentation and best practices |
| **best-practices-researcher** | Research external best practices |
| **git-history-analyzer** | Analyze git history for code evolution |
| **repo-research-analyst** | Research repository structure and patterns |

### Commands (7)

| Command | Description |
|---------|-------------|
| `/compare` | Compare screenshot to Figma design |
| `/a11y` | Run accessibility audit on a URL |
| `/check` | Run full design verification on a URL |
| `/atomic-audit` | Audit codebase for atomic design compliance |
| `/atomic-init` | Initialize atomic design system structure |
| `/atomic-migrate` | Migrate components to atomic design |
| `/agent-native-audit` | Audit for agent-native architecture principles |

## Installation

### Option 1: Clone to your home directory (global)

```bash
# Clone to ~/.claude-plugins
git clone https://github.com/cbarker95/claude-plugins.git ~/.claude-plugins

# Symlink into your global Claude config
ln -s ~/.claude-plugins/.claude/* ~/.claude/
```

### Option 2: Add as a git submodule (per-project)

```bash
# Add to your project
git submodule add https://github.com/cbarker95/claude-plugins.git .claude-plugins

# Symlink into your project's .claude directory
mkdir -p .claude
ln -s ../.claude-plugins/.claude/* .claude/
```

### Option 3: Copy what you need

```bash
# Clone temporarily
git clone https://github.com/cbarker95/claude-plugins.git /tmp/claude-plugins

# Copy specific skills/agents/commands
cp -r /tmp/claude-plugins/.claude/skills/frontend-design .claude/skills/
cp /tmp/claude-plugins/.claude/agents/design-iterator.md .claude/agents/
cp /tmp/claude-plugins/.claude/commands/a11y.md .claude/commands/
```

## Usage

Once installed, the skills, agents, and commands will be automatically available in Claude Code.

### Using Skills
Skills are loaded by Claude automatically when relevant to your task, or you can explicitly invoke them.

### Using Commands
```bash
# In Claude Code
/a11y https://your-site.com
/atomic-audit src/components
/agent-native-audit
```

### Using Agents
Agents are used automatically by Claude when appropriate. They can also be invoked via the Task tool.

## Structure

```
claude-plugins/
└── .claude/
    ├── agents/          # 13 specialized agents
    ├── commands/        # 7 slash commands
    └── skills/          # 3 comprehensive skills
        ├── atomic-design-system/
        ├── agent-native-architecture/
        └── frontend-design/
```

## License

MIT
