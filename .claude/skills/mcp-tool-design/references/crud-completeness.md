# CRUD Completeness

## The Principle

**Every entity exposed to agents must have full CRUD operations.**

If an agent can read something, it should be able to create, update, and delete it (where those operations make sense). Incomplete CRUD creates capability gaps that frustrate agents and users alike.

---

## The CRUD Matrix

For every entity in your system, map out what operations exist:

```markdown
## Entity: Project

| Operation | UI | Agent Tool | Status |
|-----------|-----|------------|--------|
| Create | New Project button | create_project | ✅ |
| Read | Project page | read_project | ✅ |
| Update | Settings panel | update_project | ✅ |
| Delete | Delete button | delete_project | ❌ MISSING |
| List | Dashboard | list_projects | ✅ |
```

### Required Operations

1. **Create** — Bring new instances into existence
2. **Read** — Retrieve a single instance by ID
3. **Update** — Modify an existing instance
4. **Delete** — Remove an instance
5. **List** — Retrieve multiple instances (with filters)

---

## Common CRUD Gaps

### 1. Missing Delete

Most common gap. Often due to fear of agents deleting things.

**Wrong:**
```typescript
// Only has create, read, update
// No way for agent to delete
```

**Right:**
```typescript
tool("delete_project", {
  id: z.string(),
}, async ({ id }) => {
  const project = await db.projects.get(id);
  if (!project) {
    return { text: `Project ${id} not found`, isError: true };
  }

  // Soft delete by default
  await db.projects.update(id, { deleted_at: new Date() });

  return {
    text: `Deleted project "${project.name}". Can be restored within 30 days.`
  };
});
```

### 2. Missing List with Filters

Can read one, can't query many.

**Wrong:**
```typescript
// Can only get by exact ID
tool("get_user", { id: z.string() }, ...)
```

**Right:**
```typescript
tool("list_users", {
  query: z.string().optional(),
  role: z.string().optional(),
  limit: z.number().default(20),
  offset: z.number().default(0),
}, async ({ query, role, limit, offset }) => {
  const users = await db.users.find({
    where: {
      ...(query && { name: { contains: query } }),
      ...(role && { role }),
    },
    take: limit,
    skip: offset,
  });

  return {
    text: `Found ${users.length} users:\n${users.map(u => `- ${u.name} (${u.role})`).join('\n')}`
  };
});
```

### 3. Partial Update Only

Can change some fields, but not others.

**Wrong:**
```typescript
// Only lets agent update name
tool("update_project", {
  id: z.string(),
  name: z.string(),
}, ...)
```

**Right:**
```typescript
tool("update_project", {
  id: z.string(),
  name: z.string().optional(),
  description: z.string().optional(),
  archived: z.boolean().optional(),
  settings: z.object({
    notifications: z.boolean().optional(),
    visibility: z.enum(["public", "private"]).optional(),
  }).optional(),
}, async ({ id, ...updates }) => {
  // Only apply provided fields
  const project = await db.projects.update(id, updates);
  return { text: `Updated project:\n${JSON.stringify(project, null, 2)}` };
});
```

---

## Design Patterns

### 1. Consistent Naming

```typescript
// Entity: project
create_project(data)
read_project(id)
update_project(id, data)
delete_project(id)
list_projects(filters?)

// Entity: member
create_member(project_id, data)
read_member(id)
update_member(id, data)
delete_member(id)
list_members(project_id, filters?)
```

### 2. Rich Responses

Every CRUD operation should return enough context:

```typescript
// Create: Return created entity
{ text: `Created project "${name}" (id: ${id})`, data: project }

// Read: Return full entity
{ text: `Project: ${name}\nDescription: ${desc}\nMembers: ${count}`, data: project }

// Update: Return before/after or just updated
{ text: `Updated project. New name: "${name}"`, data: project }

// Delete: Confirm what was deleted
{ text: `Deleted project "${name}". ${remaining} projects remain.` }

// List: Return count and items
{ text: `Found ${count} projects:\n${list}`, data: projects }
```

### 3. Soft Delete by Default

Prefer soft delete for safety, with option to hard delete:

```typescript
tool("delete_project", {
  id: z.string(),
  permanent: z.boolean().default(false),
}, async ({ id, permanent }) => {
  if (permanent) {
    await db.projects.delete(id);
    return { text: `Permanently deleted project ${id}` };
  } else {
    await db.projects.update(id, { deleted_at: new Date() });
    return { text: `Soft-deleted project ${id}. Can be restored.` };
  }
});

tool("restore_project", {
  id: z.string(),
}, async ({ id }) => {
  await db.projects.update(id, { deleted_at: null });
  return { text: `Restored project ${id}` };
});
```

---

## Nested Entities

For entities with parent-child relationships:

```typescript
// Parent: Project
create_project(data)
read_project(id)
...

// Child: Task (belongs to project)
create_task(project_id, data)     // Note: parent ID in create
read_task(id)                      // Task ID alone is sufficient
update_task(id, data)
delete_task(id)
list_tasks(project_id, filters?)  // Note: parent ID in list
```

### Bulk Operations

For collections, add bulk variants:

```typescript
// Single
create_task(project_id, data)
delete_task(id)

// Bulk
create_tasks(project_id, data[])
delete_tasks(ids[])
update_tasks(updates[])  // Array of { id, ...changes }
```

---

## CRUD Audit Checklist

When auditing CRUD completeness:

```markdown
## CRUD Audit: [Application Name]

### Entity: [Entity Name]

| Operation | Tool | Status | Notes |
|-----------|------|--------|-------|
| Create | create_X | ✅/❌ | |
| Read | read_X | ✅/❌ | |
| Update | update_X | ✅/❌ | |
| Delete | delete_X | ✅/❌ | |
| List | list_Xs | ✅/❌ | |

### Missing Operations
- [ ] Operation 1: Impact and fix
- [ ] Operation 2: Impact and fix

### Partial Operations
- [ ] update_X: Missing fields [x, y, z]
- [ ] list_X: Missing filters [a, b]
```

---

## Anti-Patterns

### 1. Read-Only Entities

```typescript
// BAD: Can see but not touch
tool("get_settings", ...) // No update_settings
```

If agents can see it, they usually need to change it.

### 2. Create-Only Entities

```typescript
// BAD: Can create but not manage
tool("create_webhook", ...) // No list, update, delete
```

Users will ask agents to manage what they created.

### 3. Implicit Operations

```typescript
// BAD: Update implicitly creates
tool("set_preference", { key, value }, ...) // Creates if not exists

// GOOD: Explicit create vs update
tool("create_preference", { key, value }, ...)
tool("update_preference", { key, value }, ...)
```

Explicit operations give agents clearer semantics.

---

## CRUD for Different Entity Types

### Configuration Entities

```typescript
// Settings are typically get/set pairs
read_settings(scope?)
update_settings(scope?, changes)

// Or key-value for flexibility
read_setting(key)
update_setting(key, value)
delete_setting(key)  // Reset to default
list_settings(prefix?)
```

### Transactional Entities

```typescript
// Orders, payments — often limited updates
create_order(data)
read_order(id)
update_order(id, { status })  // Limited fields
cancel_order(id)              // Special operation, not delete
list_orders(filters)
// No delete — transactions are immutable
```

### Temporal Entities

```typescript
// Events, logs — append-only
create_event(data)
read_event(id)
list_events(time_range, filters)
// No update or delete — history is immutable
```

---

## Summary

**CRUD Completeness ensures agents aren't arbitrarily limited.**

For every entity:
1. Map UI operations to tools
2. Fill CRUD gaps
3. Add filters to list operations
4. Return rich context in responses
5. Use soft delete for safety

The goal: If the UI lets users manage it, agents should be able to manage it too.
