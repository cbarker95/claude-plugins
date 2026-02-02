# Primitives Pattern

## Core Principle

Tools should be atomic primitives that enable capability, not workflows that encode decisions.

**The key insight:** Agent intelligence is in the prompt, not the tools. Tools provide capability. The prompt tells the agent when and how to use them.

---

## Wrong: Workflow Tools

Tools that do multiple things or make decisions:

```typescript
// BAD: Encodes business logic
tool("process_feedback", {
  feedback: z.string(),
  category: z.enum(["bug", "feature", "question"]),
  priority: z.enum(["low", "medium", "high"]),
}, async ({ feedback, category, priority }) => {
  // Tool decides how to process
  const processed = categorize(feedback);
  const stored = await saveToDatabase(processed);
  const notification = await notify(priority);
  return { processed, stored, notification };
});
```

Problems:
- Tool decides categorization logic
- Tool decides notification rules
- Agent can't customize workflow
- New requirements need code changes

---

## Right: Primitive Tools

Atomic operations that the agent composes:

```typescript
// GOOD: Primitives
tool("store_item", {
  key: z.string(),
  value: z.any(),
}, async ({ key, value }) => {
  await db.set(key, value);
  return { text: `Stored ${key}` };
});

tool("send_message", {
  channel: z.string(),
  content: z.string(),
}, async ({ channel, content }) => {
  await messenger.send(channel, content);
  return { text: "Sent" };
});

tool("read_item", {
  key: z.string(),
}, async ({ key }) => {
  const value = await db.get(key);
  return { text: value ? JSON.stringify(value) : `Not found: ${key}` };
});
```

Now the agent can:
- Categorize feedback using its judgment
- Decide priority based on content
- Choose when to notify based on context
- Compose new workflows via prompt changes

---

## Naming Principles

Tool names should describe capability, not use case:

| Wrong | Right | Why |
|-------|-------|-----|
| `process_user_feedback` | `store_item` | Tool doesn't know it's feedback |
| `create_feedback_summary` | `write_file` | Tool doesn't know content type |
| `send_notification` | `send_message` | Tool doesn't know purpose |
| `deploy_to_production` | `git_push` | Tool doesn't know environment |
| `analyze_sentiment` | `read_text` + agent | Analysis is judgment, not tool |

The prompt tells the agent *when* to use primitives. The tool just provides *capability*.

---

## Input Design

Inputs should be simple data, not decisions.

### Wrong: Tool Accepts Decisions

```typescript
tool("format_content", {
  content: z.string(),
  format: z.enum(["markdown", "html", "json"]),
  style: z.enum(["formal", "casual", "technical"]),
}, ...)
```

Problems:
- Format decision is business logic
- Style decision is judgment
- New formats need code changes

### Right: Tool Accepts Data

```typescript
tool("write_file", {
  path: z.string(),
  content: z.string(),
}, ...)
```

The agent decides:
- To write `index.html` with HTML content
- To write `data.json` with JSON content
- The style based on context

---

## Output Design

Return enough information for the agent to verify and iterate.

### Wrong: Minimal Output

```typescript
async ({ key }) => {
  await db.delete(key);
  return { text: "Deleted" };
}
```

Agent can't:
- Verify the right thing was deleted
- Know current state
- Make informed next decisions

### Right: Rich Output

```typescript
async ({ key }) => {
  const existed = await db.has(key);
  if (!existed) {
    return { text: `Key ${key} did not exist`, isError: true };
  }
  const oldValue = await db.get(key);
  await db.delete(key);
  const remaining = await db.count();
  return {
    text: `Deleted ${key} (was: ${JSON.stringify(oldValue)}). ${remaining} items remaining.`
  };
}
```

Agent can:
- Verify correct key was deleted
- Know what was lost
- Understand current state
- Make informed decisions

---

## Primitive Categories

### Storage Primitives
```typescript
store_item(key, value)
read_item(key)
list_items(prefix?, limit?)
delete_item(key)
```

### Communication Primitives
```typescript
send_message(channel, content)
read_messages(channel, limit?)
```

### File Primitives
```typescript
read_file(path)
write_file(path, content)
list_files(directory, pattern?)
delete_file(path)
```

### HTTP Primitives
```typescript
http_get(url, headers?)
http_post(url, body, headers?)
http_put(url, body, headers?)
http_delete(url, headers?)
```

### Process Primitives
```typescript
run_command(command, args?)
get_output(process_id)
stop_process(process_id)
```

---

## Progression: Primitives to Domain Tools

Start with primitives, add domain tools when patterns emerge:

### Stage 1: Pure Primitives
```typescript
// Agent uses read_file, write_file, run_command
// Flexible but verbose
```

### Stage 2: Domain Vocabulary
```typescript
// Add domain tools for common patterns
store_feedback(id, content, metadata)  // Still primitive, but domain-aware
list_feedback(filter?)
```

### Stage 3: Optimized Hot Paths
```typescript
// After observing agent patterns, optimize specific workflows
// But keep primitives available for edge cases
```

The key: Agent can always fall back to primitives for unanticipated needs.

---

## Anti-Patterns Checklist

When reviewing tools, check for:

- [ ] **Workflow encoding** — Tool does multiple operations
- [ ] **Decision making** — Tool chooses between options
- [ ] **Business logic** — Tool knows about domain rules
- [ ] **Static limitations** — Hardcoded enums instead of strings
- [ ] **Minimal outputs** — Not enough info for agent to verify
- [ ] **Imperative names** — `process_X` instead of `store_X`
