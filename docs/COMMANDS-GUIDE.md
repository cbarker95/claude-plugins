# Commands Guide

Comprehensive reference for all commands in this toolkit. Commands are invoked as `/command-name` within a Claude Code session.

---

## Quick Reference

| Command | Category | Description |
|---------|----------|-------------|
| `/agent-native-audit` | Architecture | Score codebase against 8 agent-native principles |
| `/parity-audit` | Architecture | Audit action parity between UI and agent tools |
| `/mcp-design` | Architecture | Design MCP tools for agent-native applications |
| `/generate-tests` | Architecture | Generate agent capability and parity tests |
| `/atomic-audit` | Design System | Audit components against atomic design methodology |
| `/atomic-init` | Design System | Scaffold a new atomic design system |
| `/atomic-migrate` | Design System | Migrate existing components to atomic structure |
| `/check` | Design Verification | Run full design verification on a URL |
| `/compare` | Design Verification | Compare screenshot against Figma design |
| `/a11y` | Design Verification | WCAG 2.1 AA accessibility audit |
| `/prd` | Product Strategy | Generate or update a Product Requirements Document |
| `/roadmap` | Product Strategy | Generate prioritized product roadmap |
| `/competitive` | Product Strategy | Run competitive analysis |
| `/dev` | Development | Co-development workflow with AI assistance |
| `/release` | Development | Plan and execute a software release |
| `/parallel` | Development | Manage git worktrees for parallel development |
| `/slice` | Development | Analyze codebase for vertical slicing |

---

## Commands by Category

### Architecture & Agent-Native

Commands for building, auditing, and testing agent-native applications where AI agents are first-class citizens.

### Design System

Commands for implementing and maintaining atomic design systems with token-based architecture.

### Design Verification

Commands for visual QA, accessibility auditing, and design-implementation comparison.

### Product Strategy

Commands for defining product direction, competitive positioning, and roadmap planning.

### Development & Operations

Commands for day-to-day development, releases, and parallel workstreams.

---

## Architecture & Agent-Native Commands

### `/agent-native-audit`

Run a comprehensive review of the codebase against 8 agent-native architecture principles, producing a scored report.

**Usage:**
```
/agent-native-audit                    # Full audit of all 8 principles
/agent-native-audit action parity      # Audit single principle
/agent-native-audit 3                  # Audit principle by number
```

**Arguments:** Optional principle name or number to audit a single principle.

Valid single-principle arguments:
- `action parity` or `1`
- `tools` or `primitives` or `2`
- `context` or `injection` or `3`
- `shared` or `workspace` or `4`
- `crud` or `5`
- `ui` or `integration` or `6`
- `discovery` or `7`
- `prompt` or `features` or `8`

**Execution Phases:**

1. **Load Skill** -- Loads the agent-native-architecture skill for reference material.
2. **Parallel Sub-Agents** -- Launches 8 sub-agents (one per principle), each producing a specific score in X/Y format:
   - **Action Parity** -- Enumerates all user actions, checks for corresponding agent tools.
   - **Tools as Primitives** -- Classifies each tool as primitive (good) or workflow (bad).
   - **Context Injection** -- Checks what dynamic context is injected into system prompts.
   - **Shared Workspace** -- Verifies agents and users operate on the same data stores.
   - **CRUD Completeness** -- Checks every entity for full Create/Read/Update/Delete coverage.
   - **UI Integration** -- Verifies agent actions are immediately reflected in the UI.
   - **Capability Discovery** -- Checks for 7 discovery mechanisms (onboarding, help, hints, etc.).
   - **Prompt-Native Features** -- Classifies features as prompt-defined (good) vs code-defined (bad).
3. **Compile Report** -- Aggregates scores into a summary table with status indicators and top 10 recommendations.

**Output Format:**
- Summary table with scores per principle (percentage + status indicator)
- Overall agent-native score as a percentage
- Status legend: Excellent (80%+), Partial (50-79%), Needs Work (<50%)
- Top 10 recommendations ranked by impact
- List of top 5 strengths

---

### `/parity-audit`

Audit that agents can do everything users can do through the UI, producing a parity score and gap analysis.

**Usage:**
```
/parity-audit
```

**Execution Phases:**

1. **UI Action Discovery** -- Three parallel sub-agents scan:
   - Component handlers (onClick, onSubmit, onChange) and the API calls they trigger
   - API routes and CRUD operations
   - Agent tool inventory (MCP server definitions, tool registrations)
2. **Parity Mapping** -- Creates a mapping table of UI actions to agent tools with status indicators.
3. **Gap Analysis** -- Prioritizes gaps as Critical (blocks common tasks), Important (reduces agent value), or Nice-to-Have (edge cases).
4. **Test Generation** -- Optionally generates parity test files and/or a gap report.
5. **Output** -- Writes a comprehensive audit report.

**Output Format:**
- Summary metrics (total UI actions, covered by tools, parity score, critical gaps)
- Coverage breakdown by feature area
- Full parity map (action-to-tool mapping)
- Prioritized gap analysis with tool recommendations
- Saved to: `docs/parity/audit-[date].md`

**Scoring:**
```
Parity Score = (Actions with Tools) / (Total Actions) x 100

Targets:
  100%  Full parity (ideal)
  90%+  Good (acceptable for v1)
  80-90% Moderate gaps (prioritize fixes)
  <80%  Significant gaps (agents feel limited)
```

**Related:** `/generate-tests`, `/mcp-design`

---

### `/mcp-design`

Design MCP (Model Context Protocol) tools for agent-native applications, following the primitives pattern.

**Usage:**
```
/mcp-design
```

**Execution Phases:**

1. **Discovery** -- Three parallel sub-agents analyze:
   - Entity models, schemas, relationships, and existing operations
   - UI capabilities (forms, buttons, CRUD operations exposed in the frontend)
   - Existing MCP tools, REST routes, or GraphQL schemas
2. **Gap Analysis** -- Identifies entities without full CRUD, UI actions without agent tools, and partial tool coverage. Presents gaps to user for prioritization.
3. **Tool Design** -- For each selected capability, produces a tool specification including:
   - Purpose and usage description
   - Input schema (with Zod-style types)
   - Success response format (with "next actions" guidance)
   - Error responses (Not Found, Permission Denied, Validation Error)
   - Implementation and security notes
4. **CRUD Completeness Check** -- Verifies full CRUD for each entity (Create, Read, Update, Delete, List).
5. **Output** -- Writes tool specifications to `docs/mcp/[feature]-tools.md` or `docs/mcp/server-spec.md`.

**Design Principles Applied:**
1. Primitives over workflows -- atomic operations, agent composes
2. Data over decisions -- tools accept data, not choices
3. Rich over minimal -- responses include context
4. Guidance over rejection -- errors help fix problems
5. Parity with UI -- agents can do what users can do
6. Full CRUD -- every entity has complete operations

**Related:** `/parity-audit`, `/generate-tests`, `/dev`

---

### `/generate-tests`

Generate tests that verify agent capabilities, action parity, and behavior.

**Usage:**
```
/generate-tests                     # Interactive: choose type and scope
/generate-tests --type parity       # Only parity tests
/generate-tests --scope projects    # Specific feature area
/generate-tests --scope git-diff    # Based on recent changes
```

**Execution Phases:**

1. **Test Scope Discovery** -- Asks user to select test type (Parity, Capability, Behavior, or All) and scope (full codebase, specific feature, git diff, or from PRD).
2. **Context Gathering** -- Launches sub-agents appropriate to the selected scope:
   - Parity: scans components for handlers, maps to tools, identifies gaps
   - Capability: extracts testable capabilities from PRD/requirements
   - Behavior: extracts expected behaviors from system prompts and guidelines
3. **Test Generation** -- Produces test files using three templates:
   - **Parity tests** -- Verify agents can do what users can in UI (action-to-tool mapping)
   - **Capability tests** -- Verify agents can achieve business outcomes (scenario-based)
   - **Behavior tests** -- Verify agents follow guidelines (confirmations, error handling)
4. **Test Utilities** -- Generates supporting files: test harness, custom matchers, fixture scenarios.
5. **Output** -- Writes files to `tests/parity/`, `tests/capability/`, `tests/behavior/`, plus utilities, and produces a generation report.

**Output Structure:**
```
tests/
  parity/          # Action parity tests
  capability/      # Business outcome tests
  behavior/        # Guideline compliance tests
  test-utils.ts    # Test harness
  matchers.ts      # Custom matchers
  fixtures/        # Scenario data
```

**Related:** `/parity-audit`, `/mcp-design`, `/dev`

---

## Design System Commands

### `/atomic-audit`

Run a comprehensive atomic design audit on a path, evaluating component hierarchy, token architecture, composition patterns, and documentation.

**Usage:**
```
/atomic-audit                       # Audit src/
/atomic-audit src/components        # Audit specific path
```

**Arguments:** Optional path to audit (defaults to `src/`).

**Execution Phases:**

1. **Parallel Analysis** -- Two agents run simultaneously:
   - **component-hierarchy-analyzer** -- Scans all components, classifies into atomic levels (atoms, molecules, organisms, templates, pages), identifies violations.
   - **token-auditor** -- Extracts design tokens from CSS/SCSS, evaluates naming against EightShapes taxonomy, finds hardcoded values.
2. **Scoring** -- **atomic-compliance-scorer** reads outputs from both analyzers and produces a compliance score with recommendations.

**Output Files** (created in `atomic-audit/` directory):
- `component-hierarchy.json` -- Component inventory with classifications
- `design-tokens.json` -- Token extraction and analysis
- `compliance-score.md` -- Overall score and detailed breakdown
- `recommendations.md` -- Prioritized list of improvements

**Report Format:**
- Overall score out of 100
- Four scored areas (each out of 25): Component Hierarchy, Token Architecture, Composition Patterns, Documentation
- Star ratings per area
- Hierarchy violations list
- Token issues list
- Prioritized recommendations

**References:** Reads `.claude/skills/atomic-design-system/references/evaluation-criteria.md` and `design-tokens.md` before running.

---

### `/atomic-init`

Scaffold a new atomic design system with folder structure, token files, and starter components.

**Usage:**
```
/atomic-init                        # Initialize with React (default)
/atomic-init react                  # Explicit React
/atomic-init vue                    # Vue 3 with Composition API
/atomic-init vanilla                # Plain CSS + Web Components
```

**Arguments:** Optional framework -- `react` (default), `vue`, or `vanilla`.

**What Gets Created:**
```
src/
  design-system/
    tokens/          # Color, typography, spacing, shadow CSS custom properties
      colors.css
      typography.css
      spacing.css
      shadows.css
      index.css
    atoms/           # Button, Input, Label, Icon components
    molecules/       # FormField (composes Label + Input + error)
    organisms/
    templates/
    index.{ts|js}    # Main export
  .storybook/        # Storybook config (React/Vue only)
```

**Starter Components:**
- **Atoms:** Button (4 variants, 3 sizes), Input (with error state and slots), Label (with required indicator), Icon (SVG wrapper)
- **Molecules:** FormField (composes Label + Input + error text)

**Token Defaults:** CSS custom properties following primitive/semantic naming:
- Primitives: `--color-blue-500`, `--color-gray-900`, etc.
- Semantic: `--color-text-primary`, `--color-action-primary`, `--color-feedback-error`, etc.

**Post-Init Steps:**
1. Install framework-specific dependencies
2. Set up path aliases in tsconfig/vite config
3. Configure Storybook (optional)
4. Run `/atomic-audit` to verify structure

---

### `/atomic-migrate`

Iteratively migrate existing components to atomic design structure using the Ralph Wiggum loop pattern (one task per iteration with clear completion signals).

**Usage:**
```
/atomic-migrate                             # Full migration from Phase 2
/atomic-migrate --phase=3                   # Start from Atom Extraction
/atomic-migrate src/components/Button.tsx   # Migrate single component
```

**Arguments:** Optional `--phase=N` to start at a specific phase, or a component path for single-component migration.

**Migration Phases:**

| Phase | Name | Description | Completion Signal |
|-------|------|-------------|-------------------|
| 1 | Audit | Run `/atomic-audit` (prerequisite) | Audit report generated |
| 2 | Token Extraction | Replace hardcoded values with CSS custom properties | No hardcoded hex/px in component CSS |
| 3 | Atom Extraction | Move atomic components to `atoms/` directory | All atoms in `atoms/` |
| 4 | Molecule Composition | Refactor molecules to import from `@/atoms` | All molecules in `molecules/` |
| 5 | Organism Assembly | Refactor organisms to import from atoms and molecules | All organisms in `organisms/` |
| 6 | Template Extraction | Extract layout patterns into `templates/` | All templates in `templates/` |
| 7 | Cleanup | Update all imports from legacy locations, remove re-exports | No imports from legacy locations |

**Iteration Output:** After each component migration, reports action taken, test status, visual regression status, and next component.

**Backpressure Constraints** (migration fails if violated):
- Atom imports another atom
- Molecule imports from organisms
- Hardcoded color/spacing value introduced
- Tests fail after migration
- Visual regression detected

**Checkpoint Strategy:**
- Git tag before each phase: `atomic-migration-phase-{n}-start`
- Git commit after each component: `refactor({level}): migrate {ComponentName} to atomic structure`
- Can be stopped and resumed at any point

**References:** Reads `migration-patterns.md` and `component-hierarchy.md` from the atomic design skill.

---

## Design Verification Commands

### `/check`

Run a complete design verification check on a URL, combining visual comparison, accessibility audit, and UX consistency analysis.

**Usage:**
```
/check <url>
```

**Arguments:** URL to verify (required).

**Allowed Tools:** Bash, Read, Write, WebFetch, Task

**Execution Steps:**
1. **Capture Screenshot** -- Uses Playwright to capture the current state of the URL.
2. **Fetch Design** -- Retrieves the corresponding Figma design via Figma MCP.
3. **Run Agents** (parallel):
   - Design comparison (visual diff)
   - Accessibility audit (WCAG 2.1 AA)
   - UX consistency check
4. **Generate Report** -- Compiles findings into a structured report.

**Output Format:**
- Overall score (0-100)
- Design fidelity issues (with screenshots)
- Accessibility violations
- UX recommendations
- Priority ranking of issues

---

### `/compare`

Compare a captured screenshot against its Figma design source for visual discrepancies.

**Usage:**
```
/compare <screenshot-path> <figma-url>
```

**Arguments:**
- `$1` -- Path to screenshot or URL to capture
- `$2` -- Figma file/frame URL (optional if linked in project)

**Allowed Tools:** Bash, Read, WebFetch, Task

**Analysis Points:**
1. **Layout Differences** -- Spacing, alignment, positioning
2. **Typography** -- Font sizes, weights, line heights
3. **Colors** -- Color accuracy, contrast ratios
4. **Components** -- Design system component usage
5. **Responsive** -- Breakpoint adherence

**Output Format:**
- Pixel-level differences
- Semantic differences (intentional vs bugs)
- Severity ratings for each issue

---

### `/a11y`

Run a comprehensive WCAG 2.1 AA accessibility audit on a URL.

**Usage:**
```
/a11y <url>
```

**Arguments:** URL to audit (required).

**Allowed Tools:** Bash, Read, WebFetch, Task

**Checks Performed:**

| WCAG Principle | Checks |
|---------------|--------|
| **Perceivable** | Image alt text, color contrast (4.5:1 text, 3:1 large text), text resize, captions |
| **Operable** | Keyboard navigation, focus indicators, skip links, touch targets (44x44px min) |
| **Understandable** | Form labels, error messages, consistent navigation, language attributes |
| **Robust** | Valid HTML, ARIA usage, semantic markup |

**Output Format:** Violations grouped by severity:
1. **Critical** -- Blocks users entirely
2. **Serious** -- Major barriers
3. **Moderate** -- Some difficulty
4. **Minor** -- Best practice improvements

Each violation includes code snippets and fix recommendations.

---

## Product Strategy Commands

### `/prd`

Generate or update a Product Requirements Document by analyzing the codebase, existing documentation, and user input.

**Usage:**
```
/prd
```

**Allowed Tools:** Read, Glob, Grep, Task, AskUserQuestion, Write

**Execution Phases:**

1. **Parallel Discovery** -- Three sub-agents gather:
   - Existing documentation (PRD, README, project context files)
   - Implementation state (built, partial, planned, unknown features)
   - Technical architecture (stack, patterns, dependencies, constraints)
2. **Clarification** -- Uses AskUserQuestion to resolve ambiguities around:
   - Strategic direction (target users, success metrics, positioning)
   - Scope decisions (must-have vs nice-to-have, exclusions)
   - Design philosophy (UX principles, key trade-offs)
3. **Synthesis** -- Writes a formal PRD to `docs/product/PRD.md`.

**PRD Sections:**
- Executive Summary
- Problem Statement (problem, who, evidence)
- Target Users (persona with goals, pain points, jobs-to-be-done)
- Success Metrics (with baselines and targets)
- Product Principles
- Feature Scope (Must Have / Should Have / Won't Have)
- User Flows
- Technical Constraints
- Open Questions
- Appendix

**Conversation Style:** Direct and opinionated. Challenges assumptions, suggests alternatives, asks focused questions, infers from codebase where possible.

**Related:** `/dev`, `/roadmap`, `/competitive`

---

### `/roadmap`

Generate a prioritized product roadmap from PRD, GitHub issues, current state, and dependencies.

**Usage:**
```
/roadmap
/roadmap --format timeline          # Timeline view
/roadmap --format kanban            # Kanban view
/roadmap --format theme             # Theme-based view
/roadmap --include-issues           # Link to GitHub issues
/roadmap --create-milestones        # Create GitHub milestones
```

**Options:**
- `--format [timeline|kanban|theme]` -- Output format
- `--horizon [quarter|year]` -- Planning horizon
- `--include-issues` -- Link to GitHub issues
- `--create-milestones` -- Create GitHub milestones

**Execution Phases:**

1. **Gather Inputs** -- Four parallel sub-agents analyze:
   - PRD features and priorities
   - GitHub issues (categorized: feature requests, bugs, enhancements, tech debt)
   - Current implementation state (built, in progress, not started)
   - Technical dependency map
2. **Prioritization Framework** -- User selects approach:
   - **Now/Next/Later** -- Simple, flexible
   - **RICE** -- Reach x Impact x Confidence / Effort
   - **MoSCoW** -- Must/Should/Could/Won't
   - Custom criteria
3. **Generate Roadmap** -- Produces a structured roadmap with vision, items by priority tier, dependency graph (Mermaid), risks/mitigations, success metrics, and review schedule.
4. **Interactive Refinement** -- User can move items, add/remove, adjust priorities.
5. **Output** -- Saves to `docs/product/ROADMAP.md`. Optionally creates GitHub milestones.

**Roadmap Views:**
- **Timeline** -- Quarterly breakdown with checkboxes
- **Kanban** -- Now/Next/Later columns
- **Theme** -- Grouped by product themes

**Related:** `/prd`, `/release`, `/competitive`

---

### `/competitive`

Analyze competitors to inform product positioning and roadmap priorities.

**Usage:**
```
/competitive
/competitive --competitors "Notion,Linear"
/competitive --depth deep
/competitive --focus pricing
```

**Options:**
- `--competitors [names]` -- Specific competitors to analyze
- `--depth [quick|standard|deep]` -- Analysis depth
- `--focus [features|pricing|positioning]` -- Focus area
- `--output [file]` -- Output location

**Execution Phases:**

1. **Define Scope** -- Asks user for competitors (specific names, auto-discover, or both) and analysis depth.
2. **Competitor Discovery** -- Web research and technology research to identify direct, indirect, and potential future competitors.
3. **Competitive Analysis** -- For each competitor: overview, target market, feature inventory, strengths/weaknesses, positioning, reviews/sentiment.
4. **Comparative Analysis** -- Feature matrix, positioning map (2x2), pricing comparison.
5. **Strategic Insights** -- SWOT analysis and Porter's Five Forces assessment.
6. **Recommendations** -- Differentiation opportunities, focus areas (double down, catch up, differentiate, avoid), and roadmap implications (now/next/watch).
7. **Output** -- Saves to `docs/product/competitive-analysis/` with executive summary, competitor profiles, feature matrix, SWOT, and recommendations.

**Keeping Updated:** Suggests monthly feature matrix updates, quarterly positioning reviews, and trigger-based reassessments.

**Related:** `/prd`, `/roadmap`

---

## Development & Operations Commands

### `/dev`

Co-development workflow for building products with AI assistance. Acts as a senior full-stack development partner.

**Usage:**
```
/dev
```

**Allowed Tools:** Read, Write, Edit, Glob, Grep, Bash, Task, AskUserQuestion, Skill, TodoWrite

**Startup Sequence:**

1. **Load Context** (parallel) -- Four sub-agents review:
   - PRD (product requirements, MVP must-haves, success metrics)
   - Current state (built pages/routes, APIs, components, working vs placeholder)
   - Development blog (last session's work, recommended next steps, blockers)
   - Patterns and skills (conventions from CLAUDE.md and skills)
2. **Synthesize & Plan** -- Identifies gap between PRD and current state, creates task list via TodoWrite.
3. **Clarify Intent** -- Asks user which direction to take this session.
4. **Work Loop** (Ralph Wiggum pattern -- one task per iteration):
   - Explain what's being built
   - Clarify product questions if needed
   - Build the code (parallel sub-agents when possible)
   - Test and verify
   - Commit with clear message
   - Update development blog
   - Summarize what shipped
5. **Session Wrap-up** -- Updates blog, lists next session items, notes open questions, offers to commit.

**Development Blog:** Maintains `docs/development/BLOG.md` with entries for each feature/change including what changed, why it matters, how it works, and what's next.

**Decision Framework:**
- **Ask the user:** Product decisions, UX trade-offs, scope questions, priority calls
- **Decide yourself:** Implementation details, architecture, code patterns, performance

**Related:** `/prd`, `/generate-tests`, `/release`

---

### `/release`

Plan and execute a software release with versioning, changelog, and deployment.

**Usage:**
```
/release                            # Interactive release
/release --patch --yes              # Quick patch release (skip confirmations)
/release --dry-run                  # Preview without executing
```

**Options:**
- `--major` -- Force major version bump
- `--minor` -- Force minor version bump
- `--patch` -- Force patch version bump
- `--yes` -- Skip confirmations
- `--dry-run` -- Show what would happen without executing
- `--no-push` -- Don't push to remote

**Execution Phases:**

1. **Gather Context** (parallel) -- Three sub-agents analyze:
   - Changes since last tag (commit count, features, fixes, breaking changes, contributors)
   - Test and build status (pass/fail)
   - Current version and versioning scheme
2. **Version Decision** -- Recommends MAJOR/MINOR/PATCH based on changes. Asks user to confirm.
3. **Generate Changelog** -- Parses commits, groups by category (Added, Fixed, Changed, Breaking), links issues/PRs, generates user-friendly descriptions.
4. **Create Release** -- Updates version in package.json, prepends changelog to CHANGELOG.md, commits, creates annotated git tag, pushes.
5. **Create GitHub Release** -- Uses `gh release create` with release notes.
6. **Post-Release** -- Asks about deployment, announcements, documentation updates.

**Output:** Release summary with version info, change counts, artifact checklist, and links to release, changelog, and commits.

**Related:** `/roadmap`, `/dev`

---

### `/parallel`

Manage git worktrees for multi-agent parallel development. Supports both automated agent spawning and manual terminal workflows.

**Usage:**
```
/parallel spawn                     # Create worktrees + spawn background agents
/parallel setup                     # Create worktrees (manual terminal launch)
/parallel sync                      # Sync current worktree with main
/parallel merge                     # Merge completed feature to main
/parallel merge-all                 # Merge all completed features to main
/parallel status                    # Show worktree and agent status
/parallel clean                     # Remove a worktree
```

**Subcommands:**

#### `spawn` (Automated Parallel Execution)
Creates worktrees and spawns background Task agents to work in parallel. Recommended approach.

1. Gathers context from plan file or asks user what to parallelize.
2. Creates git worktrees with dedicated branches.
3. Spawns background agents (Task tool with `run_in_background: true`) for each worktree with specific task prompts and constraints.
4. Tracks agent IDs and output files in `.parallel-agents.md`.
5. Monitors for completion (agents commit with "COMPLETE:" prefix).

#### `setup` (Manual Terminal Launch)
Creates worktrees for manual Claude instance launches.

1. Analyzes current state (fetches, verifies on main).
2. Determines slicing (uses vertical-slicer agent or asks user for 2-4 worktrees).
3. Creates worktrees with dedicated branches.
4. Generates `.parallel-dev.md` with worktree configuration, file ownership, frozen files.
5. Outputs terminal commands for launching Claude in each worktree.

#### `sync`
Synchronizes current worktree with latest main: stash, fetch, rebase, run tests, restore stash. Handles conflicts interactively.

#### `merge`
Merges a completed feature branch to main with pre-merge checklist (tests, rebase, review), confirmation, `--no-ff` merge, integration tests, and push.

#### `merge-all`
Merges all completed feature branches to main sequentially. Verifies all agents are complete, merges each branch, validates, removes worktrees, cleans up tracking file.

#### `status`
Shows status of all worktrees (branch, last commit, clean/dirty, ahead/behind) and any background agents (running/complete, commit progress).

#### `clean`
Removes a worktree and optionally its branch. Checks for uncommitted changes, confirms with user, prunes stale references.

**Error Handling:** Provides specific guidance for "branch already checked out", uncommitted changes, and merge conflicts.

**Related:** `/slice`

---

### `/slice`

Analyze a codebase and recommend how to split work for parallel development with minimal conflicts.

**Usage:**
```
/slice
```

**Allowed Tools:** Read, Write, Glob, Grep, Bash, Task, AskUserQuestion

**Execution Steps:**

1. **Understand Goals** -- Asks what needs to be parallelized (new features, refactoring, bug fixes, project phases).
2. **Analyze Codebase** (parallel) -- Three sub-agents examine:
   - Directory structure (organization, naming conventions, feature areas)
   - Feature detection (pages/routes, API endpoints, components, tests)
   - Dependency analysis (shared utilities, types, configuration, reused components)
3. **Build Feature Map** -- Synthesizes analysis into feature domains with file lists and cross-domain conflict risk.
4. **Identify Shared Code** -- Lists files that should be frozen (config, shared types, shared utilities, shared UI components).
5. **Assess Conflict Risk** -- Produces a conflict risk matrix between potential slices with specific risk details.
6. **Propose Configurations** -- Generates 2-3 options:
   - **Conservative** (2 worktrees) -- Less coordination, larger slices
   - **Balanced** (3 worktrees) -- Good parallelism/coordination balance
   - **Aggressive** (4 worktrees) -- Maximum parallelism, more coordination
7. **Present Recommendation** -- User selects configuration.
8. **Generate Slice Document** -- Writes detailed specification to `.parallel-slices.md` with file ownership, read-only dependencies, frozen files, coordination protocol, and merge order.

**Output:**
1. Console summary of recommended slicing
2. Detailed slice specification at `.parallel-slices.md`
3. Ready for `/parallel setup` to create worktrees

**Related:** `/parallel`

---

## Command Workflow Diagrams

### Product Development Lifecycle

```
/prd  -->  /competitive  -->  /roadmap  -->  /dev  -->  /release
  |                              |             |
  |                              v             v
  +------- feeds into -------> /slice --> /parallel
```

1. **Define** -- `/prd` creates the product requirements document.
2. **Research** -- `/competitive` informs positioning and priorities.
3. **Plan** -- `/roadmap` produces a prioritized roadmap from the PRD.
4. **Build** -- `/dev` implements features incrementally.
5. **Ship** -- `/release` handles versioning, changelog, and deployment.

For parallel development, `/slice` analyzes the codebase before `/parallel` creates worktrees.

### Agent-Native Architecture Workflow

```
/agent-native-audit  -->  /parity-audit  -->  /mcp-design  -->  /generate-tests
        |                       |                   |                  |
        v                       v                   v                  v
   Score all 8            Map UI actions       Design missing      Generate parity,
   principles             to agent tools       MCP tools           capability, and
                                                                   behavior tests
```

1. **Audit** -- `/agent-native-audit` scores the full architecture.
2. **Focus** -- `/parity-audit` drills into UI-to-agent action coverage.
3. **Design** -- `/mcp-design` creates specifications for missing tools.
4. **Verify** -- `/generate-tests` produces tests to validate parity and capabilities.

### Design System Workflow

```
/atomic-init  -->  /atomic-audit  -->  /atomic-migrate
     |                   |                    |
     v                   v                    v
  Scaffold          Evaluate             Migrate components
  structure         compliance           through 7 phases
```

1. **Initialize** -- `/atomic-init` scaffolds tokens, atoms, molecules.
2. **Audit** -- `/atomic-audit` scores against atomic design principles.
3. **Migrate** -- `/atomic-migrate` iteratively restructures existing components.

### Design Verification Workflow

```
/check  =  /compare  +  /a11y  +  UX consistency
   |            |           |
   v            v           v
Full design   Visual     WCAG 2.1 AA
verification  diff       compliance
```

`/check` is a composite command that runs `/compare` and `/a11y` together with UX consistency analysis.

---

## Common Patterns

### Parallel Sub-Agents

Most commands launch multiple sub-agents in parallel during their discovery phase to gather context efficiently. Results are synthesized before proceeding to the next phase.

### Ralph Wiggum Loop

Used by `/dev` and `/atomic-migrate` for iterative work: one task per iteration with clear completion signals, commits after each unit of work, and the ability to stop and resume.

### Interactive Clarification

Commands use `AskUserQuestion` to resolve ambiguity before producing output. Product decisions are deferred to the user; implementation decisions are made autonomously.

### Phased Execution

Every command follows a phased approach:
1. **Discovery/Gathering** -- Parallel analysis of codebase and context
2. **Analysis/Synthesis** -- Process findings, identify gaps
3. **User Input** -- Clarify priorities or confirm approach
4. **Generation/Execution** -- Produce artifacts or make changes
5. **Output/Reporting** -- Write reports, commit, summarize results
