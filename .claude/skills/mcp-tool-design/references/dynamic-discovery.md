# Dynamic Discovery

## The Principle

**Agents should be able to discover capabilities at runtime, not just at design time.**

Static tool lists work for simple applications. But real-world systems have:
- User-specific capabilities (permissions, roles)
- Context-dependent operations (state machines)
- Dynamically registered resources (plugins, integrations)

Agents need discovery mechanisms to adapt to runtime reality.

---

## Discovery Levels

### Level 1: Static Tools (Baseline)

```typescript
// Fixed set of tools, always available
const tools = [
  create_project,
  read_project,
  update_project,
  delete_project,
];
```

Works for: Simple applications with fixed functionality.

### Level 2: Capability Introspection

```typescript
// Tool that reveals available operations
tool("list_capabilities", {}, async () => {
  const capabilities = [
    { name: "create_project", description: "Create a new project" },
    { name: "read_project", description: "Read project details" },
    { name: "update_project", description: "Update project properties" },
    { name: "delete_project", description: "Delete a project" },
  ];

  return {
    text: `Available capabilities:\n${capabilities.map(c =>
      `- ${c.name}: ${c.description}`
    ).join('\n')}`
  };
});
```

Works for: Applications where agents need to know what's possible.

### Level 3: Context-Aware Discovery

```typescript
// Capabilities depend on user, context, state
tool("list_capabilities", {
  context: z.object({
    user_id: z.string().optional(),
    resource_id: z.string().optional(),
    resource_type: z.string().optional(),
  }).optional(),
}, async ({ context }) => {
  const user = context?.user_id
    ? await getUser(context.user_id)
    : null;

  const capabilities = [];

  // Base capabilities
  capabilities.push({ name: "list_projects", available: true });

  // Permission-gated
  if (user?.role === "admin") {
    capabilities.push({ name: "delete_project", available: true });
  } else {
    capabilities.push({ name: "delete_project", available: false, reason: "Admin only" });
  }

  // State-dependent (if checking a specific resource)
  if (context?.resource_type === "project" && context?.resource_id) {
    const project = await getProject(context.resource_id);
    if (project.status === "draft") {
      capabilities.push({ name: "publish_project", available: true });
    } else {
      capabilities.push({ name: "publish_project", available: false, reason: "Already published" });
    }
  }

  return {
    text: `Capabilities:\n${capabilities.map(c =>
      c.available
        ? `✅ ${c.name}`
        : `❌ ${c.name} (${c.reason})`
    ).join('\n')}`
  };
});
```

Works for: Permission-based systems, state machines, role-based access.

### Level 4: Self-Describing Tools

```typescript
// Each tool describes itself fully
tool("create_project", {
  data: z.object({
    name: z.string().describe("Project name (required, 1-100 chars)"),
    description: z.string().optional().describe("Project description"),
    template: z.enum(["blank", "starter", "advanced"]).optional()
      .describe("Template to start from"),
  }),
}, async ({ data }) => {
  // ... implementation
});

// Discovery tool that reads tool schemas
tool("describe_tool", {
  name: z.string(),
}, async ({ name }) => {
  const tool = tools.find(t => t.name === name);
  if (!tool) {
    return { text: `Tool "${name}" not found`, isError: true };
  }

  return {
    text: `Tool: ${tool.name}
Description: ${tool.description}
Parameters:
${formatSchema(tool.inputSchema)}`
  };
});
```

---

## Discovery Patterns

### Pattern 1: Schema Discovery

Let agents understand what inputs are valid:

```typescript
tool("get_schema", {
  entity: z.string(),
}, async ({ entity }) => {
  const schemas = {
    project: {
      name: { type: "string", required: true, maxLength: 100 },
      description: { type: "string", required: false },
      visibility: { type: "enum", values: ["public", "private"] },
    },
    task: {
      title: { type: "string", required: true },
      status: { type: "enum", values: ["todo", "in_progress", "done"] },
      assignee_id: { type: "string", required: false },
    },
  };

  const schema = schemas[entity];
  if (!schema) {
    return { text: `Unknown entity: ${entity}. Known: ${Object.keys(schemas).join(", ")}` };
  }

  return { text: `Schema for ${entity}:\n${JSON.stringify(schema, null, 2)}` };
});
```

### Pattern 2: Navigation Discovery

Let agents understand what resources exist:

```typescript
tool("list_resources", {
  type: z.string().optional(),
}, async ({ type }) => {
  const resources = await db.resources.find({
    where: type ? { type } : {},
  });

  return {
    text: `Resources:\n${resources.map(r =>
      `- ${r.type}/${r.id}: ${r.name}`
    ).join('\n')}`,
    data: resources,
  };
});

tool("describe_resource", {
  id: z.string(),
}, async ({ id }) => {
  const resource = await db.resources.get(id);
  if (!resource) {
    return { text: `Resource ${id} not found`, isError: true };
  }

  // Include available actions on this resource
  const actions = getActionsForResource(resource);

  return {
    text: `Resource: ${resource.name}
Type: ${resource.type}
Status: ${resource.status}
Available actions: ${actions.join(", ")}`,
    data: { resource, actions },
  };
});
```

### Pattern 3: Transition Discovery

For state machines, reveal valid transitions:

```typescript
tool("get_transitions", {
  resource_id: z.string(),
}, async ({ resource_id }) => {
  const resource = await db.get(resource_id);

  const transitions = {
    draft: ["publish", "archive"],
    published: ["unpublish", "archive"],
    archived: ["restore"],
  };

  const available = transitions[resource.status] || [];

  return {
    text: `Resource "${resource.name}" is ${resource.status}.
Available transitions: ${available.join(", ") || "none"}`,
  };
});
```

### Pattern 4: Integration Discovery

For systems with plugins/integrations:

```typescript
tool("list_integrations", {}, async () => {
  const integrations = await getActiveIntegrations();

  return {
    text: `Active integrations:\n${integrations.map(i =>
      `- ${i.name}: ${i.capabilities.join(", ")}`
    ).join('\n')}`,
  };
});

tool("list_integration_tools", {
  integration: z.string(),
}, async ({ integration }) => {
  const tools = await getIntegrationTools(integration);

  return {
    text: `Tools from ${integration}:\n${tools.map(t =>
      `- ${t.name}: ${t.description}`
    ).join('\n')}`,
  };
});
```

---

## Implementation Approaches

### MCP Dynamic Tool Registration

```typescript
// MCP server can add/remove tools at runtime
const server = new MCPServer();

// Initial tools
server.addTool(create_project);
server.addTool(read_project);

// Later, when user enables GitHub integration
server.addTool(github_create_issue);
server.addTool(github_list_prs);

// Or remove when disabled
server.removeTool("github_create_issue");
```

### Capability Tokens

```typescript
// Tools check capability tokens at runtime
tool("admin_operation", {
  action: z.string(),
}, async ({ action }, context) => {
  // Check runtime capabilities
  if (!context.capabilities.includes("admin")) {
    return {
      text: "This operation requires admin capability",
      isError: true,
    };
  }

  // Proceed with operation
});
```

### Resource-Scoped Tools

```typescript
// Tools are scoped to resources the agent has access to
tool("operate_on_resource", {
  resource_id: z.string(),
  operation: z.string(),
}, async ({ resource_id, operation }, context) => {
  // Check resource access
  const hasAccess = await checkAccess(context.agent_id, resource_id);
  if (!hasAccess) {
    return {
      text: `No access to resource ${resource_id}`,
      isError: true,
    };
  }

  // Check operation is valid for this resource
  const validOps = await getValidOperations(resource_id);
  if (!validOps.includes(operation)) {
    return {
      text: `Invalid operation. Valid operations: ${validOps.join(", ")}`,
      isError: true,
    };
  }

  // Execute
});
```

---

## Discovery Response Design

### Include "Next Actions"

```typescript
// After any operation, suggest what's possible next
async function createProject(data) {
  const project = await db.projects.create(data);

  return {
    text: `Created project "${project.name}"

Next actions:
- Add members: create_member(project_id: "${project.id}", ...)
- Create tasks: create_task(project_id: "${project.id}", ...)
- Update settings: update_project(id: "${project.id}", ...)`,
    data: project,
  };
}
```

### Include Constraints

```typescript
// Tell agents what's NOT possible and why
tool("delete_project", { id }, async ({ id }) => {
  const project = await db.projects.get(id);

  // Check constraints
  if (project.has_active_tasks) {
    return {
      text: `Cannot delete project with active tasks.
Resolve first: list_tasks(project_id: "${id}", status: "active")`,
      isError: true,
    };
  }

  if (project.members.length > 1) {
    return {
      text: `Cannot delete project with multiple members.
Remove members first: list_members(project_id: "${id}")`,
      isError: true,
    };
  }

  await db.projects.delete(id);
  return { text: `Deleted project "${project.name}"` };
});
```

---

## Anti-Patterns

### 1. Hidden Capabilities

```typescript
// BAD: Tool exists but agent doesn't know about it
tool("secret_admin_function", ...) // Not in list_capabilities
```

### 2. Static Error Messages

```typescript
// BAD: Doesn't tell agent what IS possible
return { text: "Operation not allowed", isError: true };

// GOOD: Guides agent to alternatives
return {
  text: "Operation not allowed for archived projects. Restore first with restore_project(id)",
  isError: true,
};
```

### 3. Outdated Discovery

```typescript
// BAD: Discovery result is stale
const capabilities = getCapabilities(); // Cached from startup
// User's permissions changed, but discovery still shows old capabilities
```

---

## Summary

Dynamic discovery enables agents to:
1. **Know what's possible** — List available tools and operations
2. **Understand context** — See what's valid for current user/state
3. **Navigate resources** — Find and understand what exists
4. **Plan ahead** — Know what operations are available next

Design discovery tools that reveal runtime reality, not just design-time intentions.
