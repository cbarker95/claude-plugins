# MCP Server Patterns

## What is MCP?

**Model Context Protocol (MCP)** is a standard for connecting AI agents to external systems. MCP servers expose:
- **Tools**: Functions the agent can call
- **Resources**: Data the agent can read
- **Prompts**: Pre-defined prompt templates

This guide covers patterns for building MCP servers that follow agent-native principles.

---

## Basic Server Structure

```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new Server({
  name: "my-mcp-server",
  version: "1.0.0",
}, {
  capabilities: {
    tools: {},
    resources: {},
  },
});

// Register tools
server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [
    {
      name: "create_item",
      description: "Create a new item",
      inputSchema: {
        type: "object",
        properties: {
          name: { type: "string", description: "Item name" },
          data: { type: "object", description: "Item data" },
        },
        required: ["name"],
      },
    },
  ],
}));

// Handle tool calls
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;

  switch (name) {
    case "create_item":
      return await createItem(args);
    default:
      throw new Error(`Unknown tool: ${name}`);
  }
});

// Start server
const transport = new StdioServerTransport();
await server.connect(transport);
```

---

## Tool Design Patterns

### Pattern 1: Primitive Tools

Keep tools atomic and composable:

```typescript
// GOOD: Atomic primitives
const tools = [
  {
    name: "store_item",
    description: "Store an item by key",
    inputSchema: {
      type: "object",
      properties: {
        key: { type: "string" },
        value: { type: "object" },
      },
      required: ["key", "value"],
    },
  },
  {
    name: "read_item",
    description: "Read an item by key",
    inputSchema: {
      type: "object",
      properties: {
        key: { type: "string" },
      },
      required: ["key"],
    },
  },
  {
    name: "delete_item",
    description: "Delete an item by key",
    inputSchema: {
      type: "object",
      properties: {
        key: { type: "string" },
      },
      required: ["key"],
    },
  },
  {
    name: "list_items",
    description: "List items with optional prefix filter",
    inputSchema: {
      type: "object",
      properties: {
        prefix: { type: "string" },
        limit: { type: "number", default: 100 },
      },
    },
  },
];
```

### Pattern 2: Rich Tool Responses

Return context, not just confirmation:

```typescript
async function handleCreateItem(args) {
  const { name, data } = args;

  // Create the item
  const item = await db.items.create({ name, data });

  // Return rich response
  return {
    content: [
      {
        type: "text",
        text: `Created item "${name}" (id: ${item.id})

Item details:
${JSON.stringify(item, null, 2)}

Next actions:
- Read: read_item(key: "${item.id}")
- Update: update_item(key: "${item.id}", value: {...})
- Delete: delete_item(key: "${item.id}")
- List all: list_items()`,
      },
    ],
  };
}
```

### Pattern 3: Error Handling

Errors should guide, not just inform:

```typescript
async function handleDeleteItem(args) {
  const { key } = args;

  const item = await db.items.get(key);

  if (!item) {
    return {
      content: [
        {
          type: "text",
          text: `Item "${key}" not found.

To find the correct key:
- list_items() to see all items
- list_items(prefix: "user-") to filter by prefix`,
        },
      ],
      isError: true,
    };
  }

  // Check dependencies
  const refs = await db.references.find({ target: key });
  if (refs.length > 0) {
    return {
      content: [
        {
          type: "text",
          text: `Cannot delete "${key}" - ${refs.length} items reference it.

Referenced by:
${refs.map(r => `- ${r.source}`).join('\n')}

Options:
1. Delete referencing items first
2. Use force_delete_item(key: "${key}", cascade: true) to delete all`,
        },
      ],
      isError: true,
    };
  }

  await db.items.delete(key);

  return {
    content: [
      {
        type: "text",
        text: `Deleted item "${key}".

${await db.items.count()} items remaining.`,
      },
    ],
  };
}
```

---

## Resource Patterns

### Pattern 1: List and Read Resources

```typescript
// List available resources
server.setRequestHandler(ListResourcesRequestSchema, async () => ({
  resources: [
    {
      uri: "project://list",
      name: "All Projects",
      description: "List of all projects",
      mimeType: "application/json",
    },
    {
      uri: "project://current",
      name: "Current Project",
      description: "Currently active project",
      mimeType: "application/json",
    },
  ],
}));

// Read resource content
server.setRequestHandler(ReadResourceRequestSchema, async (request) => {
  const { uri } = request.params;

  if (uri === "project://list") {
    const projects = await db.projects.findAll();
    return {
      contents: [
        {
          uri,
          mimeType: "application/json",
          text: JSON.stringify(projects, null, 2),
        },
      ],
    };
  }

  if (uri === "project://current") {
    const current = await db.projects.getCurrent();
    return {
      contents: [
        {
          uri,
          mimeType: "application/json",
          text: JSON.stringify(current, null, 2),
        },
      ],
    };
  }

  throw new Error(`Unknown resource: ${uri}`);
});
```

### Pattern 2: Dynamic Resource URIs

```typescript
// Resources with dynamic IDs
server.setRequestHandler(ListResourcesRequestSchema, async () => {
  const projects = await db.projects.findAll();

  return {
    resources: projects.map(p => ({
      uri: `project://${p.id}`,
      name: p.name,
      description: `Project: ${p.name}`,
      mimeType: "application/json",
    })),
  };
});

server.setRequestHandler(ReadResourceRequestSchema, async (request) => {
  const { uri } = request.params;

  const match = uri.match(/^project:\/\/(.+)$/);
  if (match) {
    const project = await db.projects.get(match[1]);
    if (!project) {
      throw new Error(`Project not found: ${match[1]}`);
    }
    return {
      contents: [
        {
          uri,
          mimeType: "application/json",
          text: JSON.stringify(project, null, 2),
        },
      ],
    };
  }

  throw new Error(`Unknown resource URI format: ${uri}`);
});
```

---

## Prompt Patterns

### Pattern 1: Task-Specific Prompts

```typescript
server.setRequestHandler(ListPromptsRequestSchema, async () => ({
  prompts: [
    {
      name: "analyze_project",
      description: "Analyze a project and suggest improvements",
      arguments: [
        {
          name: "project_id",
          description: "ID of the project to analyze",
          required: true,
        },
      ],
    },
    {
      name: "generate_report",
      description: "Generate a status report",
      arguments: [
        {
          name: "format",
          description: "Report format: summary, detailed, or executive",
          required: false,
        },
      ],
    },
  ],
}));

server.setRequestHandler(GetPromptRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;

  if (name === "analyze_project") {
    const project = await db.projects.get(args.project_id);

    return {
      messages: [
        {
          role: "user",
          content: {
            type: "text",
            text: `Analyze this project and suggest improvements:

Project: ${project.name}
Status: ${project.status}
Created: ${project.created_at}
Tasks: ${project.task_count} (${project.completed_count} completed)

Details:
${JSON.stringify(project, null, 2)}

Please provide:
1. Current status assessment
2. Potential risks or blockers
3. Recommendations for improvement
4. Suggested next actions`,
          },
        },
      ],
    };
  }

  throw new Error(`Unknown prompt: ${name}`);
});
```

---

## Server Configuration Patterns

### Pattern 1: Environment-Based Configuration

```typescript
const config = {
  database: process.env.DATABASE_URL,
  apiKey: process.env.API_KEY,
  debug: process.env.DEBUG === "true",
};

// Validate required config
const required = ["database", "apiKey"];
for (const key of required) {
  if (!config[key]) {
    console.error(`Missing required config: ${key}`);
    process.exit(1);
  }
}
```

### Pattern 2: Capability-Based Tool Registration

```typescript
// Register tools based on available capabilities
const tools = [];

// Core tools always available
tools.push(list_items, read_item, create_item);

// Admin tools only with admin token
if (config.adminToken) {
  tools.push(delete_item, purge_items, admin_settings);
}

// Integration tools only if configured
if (config.githubToken) {
  tools.push(github_create_issue, github_list_prs);
}

if (config.slackWebhook) {
  tools.push(slack_send_message, slack_create_channel);
}

server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools,
}));
```

### Pattern 3: Authentication and Context

```typescript
// Pass context to tool handlers
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;

  // Extract context from request (implementation depends on transport)
  const context = {
    userId: request.meta?.userId,
    sessionId: request.meta?.sessionId,
    permissions: await getPermissions(request.meta?.userId),
  };

  // Pass context to handler
  const handler = toolHandlers[name];
  if (!handler) {
    throw new Error(`Unknown tool: ${name}`);
  }

  return await handler(args, context);
});

// Tool can use context
async function handleDeleteProject(args, context) {
  if (!context.permissions.includes("delete:project")) {
    return {
      content: [{ type: "text", text: "Permission denied: delete:project required" }],
      isError: true,
    };
  }

  // Proceed with delete
}
```

---

## Testing MCP Servers

### Pattern 1: Tool Unit Tests

```typescript
import { describe, it, expect } from "vitest";

describe("create_item tool", () => {
  it("creates item with valid input", async () => {
    const result = await handleCreateItem({
      name: "test-item",
      data: { foo: "bar" },
    });

    expect(result.content[0].text).toContain("Created item");
    expect(result.isError).toBeUndefined();
  });

  it("returns error for missing name", async () => {
    const result = await handleCreateItem({
      data: { foo: "bar" },
    });

    expect(result.isError).toBe(true);
    expect(result.content[0].text).toContain("name is required");
  });
});
```

### Pattern 2: Integration Tests

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";

describe("MCP Server Integration", () => {
  let client: Client;

  beforeAll(async () => {
    client = new Client({ name: "test-client", version: "1.0.0" });
    await client.connect(serverTransport);
  });

  it("lists available tools", async () => {
    const result = await client.listTools();
    expect(result.tools).toContainEqual(
      expect.objectContaining({ name: "create_item" })
    );
  });

  it("creates and reads item", async () => {
    // Create
    const createResult = await client.callTool({
      name: "create_item",
      arguments: { name: "test", data: {} },
    });
    expect(createResult.content[0].text).toContain("Created");

    // Read back
    const readResult = await client.callTool({
      name: "read_item",
      arguments: { key: "test" },
    });
    expect(readResult.content[0].text).toContain("test");
  });
});
```

---

## Deployment Patterns

### Pattern 1: Stdio Transport (Local)

```typescript
// For local development and CLI usage
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const transport = new StdioServerTransport();
await server.connect(transport);
```

### Pattern 2: SSE Transport (Remote)

```typescript
// For web-based clients
import { SSEServerTransport } from "@modelcontextprotocol/sdk/server/sse.js";
import express from "express";

const app = express();

app.get("/mcp", async (req, res) => {
  const transport = new SSEServerTransport("/mcp", res);
  await server.connect(transport);
});

app.listen(3000);
```

### Pattern 3: Docker Deployment

```dockerfile
FROM node:20-alpine

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

CMD ["node", "dist/server.js"]
```

---

## Summary

Building effective MCP servers:

1. **Tools as primitives** — Atomic, composable operations
2. **Rich responses** — Context, not just confirmation
3. **Helpful errors** — Guide agents to solutions
4. **Dynamic resources** — Reflect runtime state
5. **Task prompts** — Pre-built for common operations
6. **Capability-based** — Register tools based on config
7. **Testable** — Unit and integration tests
8. **Deployable** — Multiple transport options

The goal: Build servers that make agents capable and informed.
