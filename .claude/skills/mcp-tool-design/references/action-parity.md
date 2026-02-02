# Action Parity

## The Discipline

**Whatever users can do through the UI, agents must be able to achieve through tools.**

This is the foundational principle of agent-native design. Without parity, you're building an agent that disappoints users by saying "I can't do that" when they can clearly do it themselves.

---

## Why Parity Matters

### Without Parity
```
User: "Delete that old project"
Agent: "I don't have a delete tool. Please delete it manually in the UI."
User: *frustrated* "But I can delete it right here..."
```

### With Parity
```
User: "Delete that old project"
Agent: *uses delete_project tool* "Done. Deleted 'old-project'. You now have 5 projects."
```

---

## Parity is Outcomes, Not Buttons

Parity doesn't mean 1:1 tool-to-button mapping. It means equivalent outcomes.

### Example: Publishing a Post

**UI Flow:**
1. Click "Edit"
2. Make changes
3. Click "Preview"
4. Click "Publish"
5. Confirm in modal

**Agent Tools Needed:**
```typescript
read_post(id)
update_post(id, content)
publish_post(id)
```

The agent doesn't need a `preview_post` tool — it can show the user the content directly. The agent doesn't need a `confirm_publish` tool — it can ask the user in conversation.

**Parity achieved:** User can publish via UI, agent can publish via tools.

---

## The Capability Map

Maintain a map of UI actions to agent tools:

```markdown
## Capability Map: Project Management

| UI Action | Agent Tool | Status |
|-----------|------------|--------|
| Create project | create_project | ✅ |
| View project | read_project | ✅ |
| Edit project | update_project | ✅ |
| Delete project | delete_project | ✅ |
| Archive project | update_project(archived: true) | ✅ |
| Invite member | add_member | ✅ |
| Remove member | remove_member | ❌ MISSING |
| Change role | update_member | ❌ MISSING |
| Export data | export_project | ❌ MISSING |
```

### Parity Gaps
- Remove member: Users can, agent can't
- Change role: Users can, agent can't
- Export data: Users can, agent can't

### Action Items
1. Add `remove_member` tool
2. Add `update_member` tool
3. Add `export_project` tool

---

## Gates and Restrictions

When to restrict agent capabilities:

### DO Restrict
- **Security critical:** Admin operations, billing changes, account deletion
- **Irreversible damage:** Bulk deletes without confirmation
- **Legal/compliance:** Actions requiring human approval

### DON'T Restrict
- **Normal operations:** CRUD on user's own data
- **Convenience fears:** "Users might not want agent to do this"
- **Implementation laziness:** "We didn't build a tool for that"

**Default:** If users can do it, agents should be able to do it.

**Gate pattern:** For sensitive operations, use confirmation tools:

```typescript
// Agent requests deletion
request_delete_account(reason)  // Returns confirmation_token

// User confirms in UI or agent asks for confirmation
confirm_delete_account(confirmation_token)
```

---

## Automated Parity Testing

Test that every UI action has an agent equivalent:

```typescript
// parity.test.ts
describe('Action Parity', () => {
  const uiActions = extractUIActions();  // From UI code
  const agentTools = getAgentTools();    // From MCP server

  for (const action of uiActions) {
    it(`${action.name} has agent equivalent`, () => {
      const tool = findMatchingTool(action, agentTools);
      expect(tool).toBeDefined();

      // Verify tool can achieve same outcome
      const uiResult = await performUIAction(action);
      const toolResult = await performToolAction(tool, action.params);
      expect(toolResult.outcome).toEqual(uiResult.outcome);
    });
  }
});
```

---

## Audit Workflow

When auditing parity, follow this process:

### 1. List UI Actions
```markdown
## UI Actions Inventory

### Projects
- Create project
- View project
- Edit project settings
- Delete project
- Archive/unarchive project
- Duplicate project

### Members
- Invite member
- Remove member
- Change role
- View member list
```

### 2. Map to Tools
```markdown
## Tool Mapping

| UI Action | Tool | Notes |
|-----------|------|-------|
| Create project | create_project | ✅ |
| View project | read_project | ✅ |
| Edit settings | update_project | ✅ |
| Delete project | ❌ | MISSING |
| Archive | update_project | archived field |
| Duplicate | ❌ | MISSING |
```

### 3. Identify Gaps
```markdown
## Parity Gaps

### Critical (Block User Tasks)
- delete_project: Users can delete, agent can't

### Important (Reduce Agent Value)
- duplicate_project: Common user action

### Nice-to-have (Edge Cases)
- bulk_archive: Rarely used
```

### 4. Prioritize Fixes
```markdown
## Action Plan

1. Add delete_project (P0 - blocks user tasks)
2. Add duplicate_project (P1 - common action)
3. Consider bulk_archive (P2 - if usage grows)
```

---

## Parity Score

Calculate a parity score for your application:

```
Parity Score = (UI Actions with Tool Equivalent) / (Total UI Actions) × 100
```

**Targets:**
- 100%: Full parity (ideal for agent-native apps)
- 90%+: Good parity (acceptable)
- <90%: Parity gaps (agents feel limited)

---

## Common Parity Gaps

### 1. Delete Operations
UI has delete, but no delete tool. Often due to fear of agents deleting things.

**Fix:** Add delete tool with confirmation or soft-delete.

### 2. Bulk Operations
UI has "select all" + action, but no bulk tools.

**Fix:** Tools should accept arrays: `delete_items(ids: string[])`

### 3. Settings/Preferences
UI has settings panel, but no way for agent to read/write settings.

**Fix:** Add `read_settings()`, `update_settings(key, value)`

### 4. Export/Import
UI has export/import, but no tools.

**Fix:** Add `export_data(format)`, `import_data(data)`

### 5. Search/Filter
UI has search, but agent can only list all items.

**Fix:** Add filter params: `list_items(query?, filters?)`
