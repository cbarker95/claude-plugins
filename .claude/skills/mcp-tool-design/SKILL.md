---
name: mcp-tool-design
description: Design MCP tools for agent-native applications. Use when building tools that agents will use, creating MCP servers, or auditing existing tools for agent compatibility. Focuses on primitives pattern, action parity, and dynamic capability discovery.
---

<why_now>
## Why MCP Tool Design Matters

MCP (Model Context Protocol) tools are how agents interact with the world. Bad tool design creates "dumb agents" — agents that can only do exactly what you coded. Good tool design creates "smart agents" — agents that can accomplish things you never anticipated.

**The key insight:** Tools provide capability. Judgment stays in prompts.

When tools encode business logic, you limit the agent to your imagination. When tools are primitives, the agent can compose them in ways you never considered.

As the agent-native architecture principle states: "Whatever a user can do, the agent should be able to do."
</why_now>

<core_principles>
## Core Principles

### 1. Tools Are Primitives, Not Workflows

Tools should be atomic operations, not multi-step processes.

**Wrong:** `process_feedback(feedback, category, priority)` — tool decides how to process
**Right:** `store_item(key, value)` + `send_message(channel, content)` — agent decides workflow

**Test:** Does this tool make decisions, or enable them?

### 2. Action Parity

Whatever users can do through UI, agents must be able to achieve through tools.

- Not 1:1 tool-to-button mapping
- Equivalent outcomes through composable primitives
- Without parity: Users ask agents to do things they can't

**Test:** Pick any UI action. Can the agent achieve the same outcome?

### 3. CRUD Completeness

Every entity type needs full CRUD (Create, Read, Update, Delete).

Incomplete CRUD breaks parity. If users can delete, agents must delete.

```typescript
// COMPLETE CRUD for "document" entity
read_document(id)
list_documents(filter?)
create_document(content)
update_document(id, content)
delete_document(id)
```

**Test:** For each entity, do all four operations exist?

### 4. Dynamic Capability Discovery

For external APIs, use discovery instead of static enums.

**Wrong:** Hardcode capability list
```typescript
category: z.enum(["health", "fitness", "sleep"])  // Breaks when API adds "nutrition"
```

**Right:** Runtime discovery
```typescript
list_available_capabilities()  // Returns current API capabilities
read_data(capability_name: string)  // Accepts any valid capability
```

**Test:** If the external API adds a feature, can agents use it without code changes?

### 5. Rich Outputs

Return enough information for agents to verify and iterate.

**Wrong:** `return { text: "Done" }`
**Right:** `return { text: "Created document abc123. 47 documents total. Size: 2.3KB" }`
</core_principles>

<intake>
## What MCP tool design help do you need?

1. **Design new tools** — Create MCP tools following primitives pattern
2. **Audit existing tools** — Check tools for parity, CRUD completeness, workflow anti-patterns
3. **Build MCP server** — Scaffold a complete MCP server with proper patterns
4. **Map capabilities** — Map UI actions to required agent tools
5. **Dynamic discovery** — Implement capability discovery for external APIs
6. **Review tool signatures** — Check inputs/outputs follow best practices

**Wait for response before proceeding.**
</intake>

<routing>
| Response | Action |
|----------|--------|
| 1, "design", "new", "create" | Read [primitives-pattern.md](./references/primitives-pattern.md), design tools |
| 2, "audit", "review", "check" | Read [action-parity.md](./references/action-parity.md) and [crud-completeness.md](./references/crud-completeness.md), run audit |
| 3, "server", "mcp server", "scaffold" | Read [mcp-server-patterns.md](./references/mcp-server-patterns.md), scaffold server |
| 4, "map", "ui", "capability" | Read [action-parity.md](./references/action-parity.md), create capability map |
| 5, "dynamic", "discovery", "external" | Read [dynamic-discovery.md](./references/dynamic-discovery.md), implement discovery |
| 6, "signature", "inputs", "outputs" | Read [primitives-pattern.md](./references/primitives-pattern.md), review signatures |

**After reading references, apply patterns to the user's specific context.**
</routing>

<tool_audit>
## Tool Audit Process

When auditing existing tools, check each against these criteria:

### 1. Primitives Check
```
[ ] Tool does ONE thing (atomic)
[ ] Tool name describes capability, not use case
[ ] Tool doesn't encode business logic
[ ] Tool doesn't make decisions (agent decides)
```

### 2. Parity Check
```
[ ] Every UI action has a tool equivalent
[ ] Agent can achieve same outcomes as users
[ ] No artificial limitations on agent
```

### 3. CRUD Check
```
[ ] Create operation exists
[ ] Read operation exists (single + list)
[ ] Update operation exists
[ ] Delete operation exists
```

### 4. Signature Check
```
[ ] Inputs are simple data, not decisions
[ ] Outputs are rich and informative
[ ] Errors are descriptive (agent can recover)
```

Output format:
```markdown
## Tool Audit: [Server Name]

### Tools Analyzed: X

### Primitives Score: X/10
- [Tool] ✅ Atomic
- [Tool] ❌ Encodes workflow

### Parity Score: X/10
- [UI Action] ✅ Has tool
- [UI Action] ❌ No agent equivalent

### CRUD Score: X/10
| Entity | C | R | U | D |
|--------|---|---|---|---|
| Doc    | ✅ | ✅ | ✅ | ❌ |

### Recommendations
1. [High priority fix]
2. [Medium priority fix]
```
</tool_audit>

<reference_index>
## Reference Files

All references in `references/`:

**Core Patterns:**
- [primitives-pattern.md](./references/primitives-pattern.md) — Tools as atomic operations, naming, signatures
- [action-parity.md](./references/action-parity.md) — Ensuring agents can do what users can do
- [crud-completeness.md](./references/crud-completeness.md) — Full CRUD for every entity
- [dynamic-discovery.md](./references/dynamic-discovery.md) — Runtime capability discovery

**Implementation:**
- [mcp-server-patterns.md](./references/mcp-server-patterns.md) — Building MCP servers with SDK
</reference_index>

<commands>
## Available Commands

- `/mcp-design` — Interactive MCP tool design workflow
- `/parity-audit` — Audit action parity between UI and agent tools
</commands>

<anti_patterns>
## Anti-Patterns

### Workflow Tools
**Wrong:** Tool that does multiple things
```typescript
process_order(items, payment, shipping) // Creates order, charges card, schedules delivery
```
**Right:** Separate primitives
```typescript
create_order(items)
charge_payment(order_id, payment)
schedule_delivery(order_id, address)
```

### Decision Tools
**Wrong:** Tool accepts decisions
```typescript
format_content(content, format: "markdown" | "html" | "json")
```
**Right:** Tool accepts data
```typescript
write_file(path, content)  // Agent decides format based on path/content
```

### Static Capabilities
**Wrong:** Hardcoded enum
```typescript
data_type: z.enum(["steps", "heart_rate", "sleep"])
```
**Right:** Dynamic discovery
```typescript
list_data_types()  // Returns available types from API
read_data(type: string)  // Accepts any valid type
```

### Minimal Outputs
**Wrong:** No context for agent
```typescript
return { text: "Done" }
```
**Right:** Rich context
```typescript
return { text: `Created ${id}. Total: ${count}. Next: ${suggestion}` }
```
</anti_patterns>

<success_criteria>
## Success Criteria

Your MCP tools are well-designed when:

### Structure
- [ ] Every tool does ONE thing (atomic primitive)
- [ ] Tool names describe capability, not use case
- [ ] No tools encode business logic or decisions
- [ ] CRUD complete for every entity type

### Parity
- [ ] Every UI action has tool equivalent
- [ ] Agent can achieve same outcomes as users
- [ ] No artificial agent limitations

### Discoverability
- [ ] External API capabilities discovered at runtime
- [ ] New API features work without code changes
- [ ] Agent can introspect available tools

### Outputs
- [ ] Rich outputs with context
- [ ] Errors are recoverable
- [ ] Agent can verify success

### The Ultimate Test

**Can an agent accomplish something you didn't anticipate?**
**If the external API adds a feature, does the agent get it automatically?**

If yes, you've built good MCP tools.
</success_criteria>
