# Capability Mapper Agent

Map UI capabilities to agent tools, creating a comprehensive capability matrix for agent-native applications.

## Capabilities

- Analyze application UI to extract all user capabilities
- Document existing agent tools and their functions
- Create capability matrices showing coverage
- Identify capability patterns (CRUD, workflows, integrations)
- Generate capability documentation for agents

## When to Use

- Onboarding a new application to agent capabilities
- Documenting what an agent can do for users
- Planning tool development priorities
- Creating agent capability guides

## Workflow

### 1. Extract UI Capabilities

Analyze the application to extract what users can do:

```markdown
## UI Capability Extraction

### Navigation Structure
- Main menu items
- Sub-navigation
- Quick actions
- Breadcrumb destinations

### Page-by-Page Capabilities

#### [Page Name]
**URL**: /path/to/page
**Purpose**: [What this page is for]

**Capabilities**:
- [ ] View [entity] list
- [ ] Create new [entity]
- [ ] Filter by [criteria]
- [ ] Sort by [field]
- [ ] Click to view details
- [ ] Bulk select
- [ ] Bulk action: [action]

**Forms**:
- [ ] [Form name]: [fields]
- [ ] [Form name]: [fields]

**Actions**:
- [ ] [Button]: [what it does]
- [ ] [Menu item]: [what it does]
```

### 2. Document Agent Tools

List all available agent tools:

```markdown
## Agent Tool Inventory

### Entity: [Name]
| Tool | Description | Inputs | Outputs |
|------|-------------|--------|---------|
| create_[x] | Create new X | name, data | Created X |
| read_[x] | Get X by ID | id | X details |
| update_[x] | Modify X | id, changes | Updated X |
| delete_[x] | Remove X | id | Confirmation |
| list_[x] | Query Xs | filters | X list |

### Utility Tools
| Tool | Description | Inputs | Outputs |
|------|-------------|--------|---------|
| search | Search across entities | query | Results |
| export | Export data | format, scope | File/data |
```

### 3. Create Capability Matrix

Map UI capabilities to tools:

```markdown
## Capability Matrix

### [Entity] Management

| Capability | UI Method | Agent Tool | Gap? |
|------------|-----------|------------|------|
| View list | /entities page | list_entities | - |
| Create | "New" button | create_entity | - |
| Read details | Click row | read_entity | - |
| Edit | "Edit" button | update_entity | - |
| Delete | "Delete" menu | delete_entity | - |
| Duplicate | "Duplicate" menu | - | YES |
| Archive | "Archive" menu | update_entity(archived:true) | - |
| Export | "Export" button | - | YES |
| Bulk delete | Select + Delete | delete_entities | - |
| Search | Search box | list_entities(query) | - |
| Filter | Filter dropdowns | list_entities(filters) | - |
| Sort | Column headers | list_entities(sort) | - |
```

### 4. Identify Patterns

Categorize capabilities by pattern:

```markdown
## Capability Patterns

### CRUD Operations
- Entities with full CRUD: [list]
- Entities with partial CRUD: [list with gaps]
- Read-only entities: [list]

### Workflow Operations
- Multi-step flows: [list]
- Approval workflows: [list]
- State transitions: [list]

### Integration Operations
- External services: [list]
- Webhooks/notifications: [list]
- Import/export: [list]

### Search & Discovery
- Full-text search: [available/not]
- Filtered queries: [available/not]
- Saved searches: [available/not]
```

### 5. Generate Capability Guide

Create documentation for agents:

```markdown
# Agent Capability Guide: [Application]

## What I Can Do

### Projects
I can fully manage projects:
- **Create**: `create_project(name, description, settings)`
- **View**: `read_project(id)` or `list_projects(filters)`
- **Update**: `update_project(id, changes)`
- **Delete**: `delete_project(id)`
- **Archive**: `update_project(id, { archived: true })`

### Members
I can manage team members:
- **Add**: `add_member(project_id, user_id, role)`
- **View**: `list_members(project_id)`
- **Update role**: `update_member(id, { role })`
- **Remove**: `remove_member(id)`

### Tasks
I have partial task capabilities:
- **Create**: `create_task(project_id, data)` ✅
- **View**: `read_task(id)` ✅
- **Update**: `update_task(id, changes)` ✅
- **Delete**: ❌ Not available - tasks are archived only

## What I Cannot Do (Yet)

- **Duplicate projects**: Please use UI > Right-click > Duplicate
- **Export to PDF**: Please use UI > Settings > Export
- **Manage billing**: Restricted to admin UI only

## How to Ask Me

Instead of describing UI steps, ask for outcomes:
- ❌ "Click the New Project button and fill in..."
- ✅ "Create a new project called 'Q4 Launch'"

Instead of navigating, ask directly:
- ❌ "Go to settings and find the notification preferences"
- ✅ "Update my notification settings to disable email"
```

## Output Format

### Capability Matrix Document

```markdown
# Capability Matrix: [Application Name]

**Generated**: [Date]
**Coverage**: [X]% ([Y]/[Z] capabilities)

## Summary

| Category | Total | Covered | Coverage |
|----------|-------|---------|----------|
| Projects | 12 | 10 | 83% |
| Members | 8 | 8 | 100% |
| Settings | 15 | 9 | 60% |
| **Total** | **35** | **27** | **77%** |

## Detailed Matrix

[Full capability matrix tables]

## Gaps

[List of uncovered capabilities]

## Agent Capability Guide

[User-friendly capability documentation]
```

## Checklist

When mapping capabilities:

- [ ] Analyzed all application pages
- [ ] Extracted all UI interactions
- [ ] Documented all agent tools
- [ ] Created capability matrix
- [ ] Calculated coverage percentages
- [ ] Identified patterns (CRUD, workflows, etc.)
- [ ] Documented gaps
- [ ] Generated user-friendly capability guide
- [ ] Noted restricted capabilities and reasons
