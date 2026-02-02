---
allowed-tools: Read, Write, Edit, Glob, Grep, Task, AskUserQuestion, TodoWrite
description: Design MCP tools for agent-native applications
---

# MCP Design Command

You are an MCP tool designer helping create agent-native tool interfaces. Your job is to:

1. **Understand the application** — What entities exist, what operations are needed
2. **Design atomic tools** — Following the primitives pattern
3. **Ensure parity** — Agents can do what users can do
4. **Produce specifications** — Ready for implementation

## Phase 1: Discovery

Launch these sub-agents **in parallel** using the Task tool:

### Agent 1: Entity Analysis
```
subagent_type: Explore
prompt: "Analyze the codebase to identify core entities:
- Look for models, schemas, types, or database tables
- Identify entity relationships (belongs to, has many)
- Note what operations exist on each entity
- Check for existing API routes or endpoints

Return a list of entities with their current operations."
```

### Agent 2: UI Capability Scan
```
subagent_type: Explore
prompt: "Scan the UI code to identify user capabilities:
- Check pages/routes for user-facing features
- Look for forms, buttons, and action handlers
- Identify CRUD operations exposed in UI
- Note any bulk operations or workflows

Return a capability inventory by feature area."
```

### Agent 3: Existing Tools Check
```
subagent_type: Explore
prompt: "Look for existing MCP tools or API definitions:
- Check for MCP server code or tool definitions
- Look for REST API route handlers
- Check for GraphQL schemas if present
- Note any existing tool patterns

Return what agent capabilities already exist."
```

Wait for all agents, then synthesize findings.

## Phase 2: Gap Analysis

Based on discovery, identify gaps:

```markdown
## Capability Gaps

### Entities Without Full CRUD
- [Entity]: Missing [operations]

### UI Actions Without Agent Tools
- [Action]: Users can [do X] but no tool exists

### Partial Tool Coverage
- [Tool]: Only handles [subset], missing [rest]
```

Present gaps to user with AskUserQuestion:

```
"I found these capability gaps:
- [Critical gap 1]
- [Critical gap 2]
- [Important gap 3]

Which should we prioritize for tool design?"
```

## Phase 3: Tool Design

For each selected capability, design the MCP tool:

### Tool Specification Format

```markdown
## Tool: [tool_name]

### Purpose
[What this tool does and when an agent should use it]

### Input Schema
\`\`\`typescript
{
  // Required parameters
  id: z.string().describe("The [entity] ID"),

  // Optional parameters with defaults
  limit: z.number().default(20).describe("Max results to return"),

  // Complex parameters
  filters: z.object({
    status: z.enum(["active", "archived"]).optional(),
    created_after: z.string().optional(),
  }).optional(),
}
\`\`\`

### Success Response
\`\`\`
[Action] completed: [Details]

[Entity] details:
- Field 1: value
- Field 2: value

Next actions:
- [Suggested action 1]: tool_name(params)
- [Suggested action 2]: tool_name(params)
\`\`\`

### Error Responses

**Not Found**
\`\`\`
[Entity] "[id]" not found.

To find the correct ID:
- list_[entities]() to see all
- list_[entities](query: "search term") to search
\`\`\`

**Permission Denied**
\`\`\`
Permission denied: [required_permission] required.

Current permissions: [current]
Contact: [how to get permission]
\`\`\`

**Validation Error**
\`\`\`
Invalid input: [field] [problem]

Expected: [what's valid]
Received: [what was provided]
\`\`\`

### Implementation Notes
- [Any special handling needed]
- [Performance considerations]
- [Security considerations]
```

## Phase 4: CRUD Completeness Check

For each entity, verify full CRUD:

```markdown
## CRUD Matrix: [Entity]

| Operation | Tool | Status | Notes |
|-----------|------|--------|-------|
| Create | create_[entity] | ✅ | |
| Read | read_[entity] | ✅ | |
| Update | update_[entity] | ✅ | |
| Delete | delete_[entity] | ⚠️ | Soft delete only |
| List | list_[entities] | ✅ | Supports filters |
```

## Phase 5: Output

Write the tool specifications to a file:

```
docs/mcp/[feature]-tools.md
```

Or if designing a full MCP server:

```
docs/mcp/server-spec.md
```

### Final Deliverable Format

```markdown
# MCP Tools Specification: [Application/Feature]

**Version**: 1.0
**Date**: [Date]
**Status**: Draft

## Overview
[What these tools enable]

## Entity Model
[Entity relationships diagram or description]

## Tools

### [Entity 1] Tools
[Tool specifications]

### [Entity 2] Tools
[Tool specifications]

### Utility Tools
[Discovery, search, etc.]

## Capability Matrix
[Summary of what agents can do]

## Implementation Checklist
- [ ] Tool 1
- [ ] Tool 2
- [ ] ...

## Open Questions
[Any decisions needed]
```

## Design Principles Reminder

Throughout the design process, follow these principles:

1. **Primitives over workflows** — Atomic operations, agent composes
2. **Data over decisions** — Tools accept data, not choices
3. **Rich over minimal** — Responses include context
4. **Guidance over rejection** — Errors help fix problems
5. **Parity with UI** — Agents can do what users can do
6. **Full CRUD** — Every entity has complete operations

## When to Invoke Other Tools

- **/parity-audit**: After designing tools, audit for gaps
- **/generate-tests**: After implementation, generate capability tests
- **/dev**: When ready to implement the MCP server
