---
name: mcp-integration
description: "Model Context Protocol (MCP) for connecting external tools and services. Use when adding integrations like databases, GitHub, Figma, Notion, or custom APIs."
---

# MCP Integration

Model Context Protocol (MCP) connects Claude Code to external tools and services.

## Quick Start

### Add an MCP Server

```bash
claude mcp add notion npx -y @anthropic-ai/mcp-notion
```

Format: `claude mcp add <name> <command> [args...]`

### List Servers

```bash
claude mcp list
```

### View Status in Session

```
/mcp
```

## Common Integrations

### GitHub

```bash
claude mcp add github npx -y @anthropic-ai/mcp-github \
  --env GITHUB_PERSONAL_ACCESS_TOKEN=ghp_xxxx
```

Then: "List open issues in my-repo"

### Filesystem (Extended Access)

```bash
claude mcp add filesystem npx -y @anthropic-ai/mcp-filesystem \
  /path/to/documents
```

### Databases

**PostgreSQL:**
```bash
claude mcp add postgres npx -y @anthropic-ai/mcp-postgres \
  "postgresql://user:pass@host:5432/db"
```

**SQLite:**
```bash
claude mcp add sqlite npx -y @anthropic-ai/mcp-sqlite \
  --db-path /path/to/database.sqlite
```

### Web/API

**Fetch:**
```bash
claude mcp add fetch npx -y @anthropic-ai/mcp-fetch
```

**Puppeteer:**
```bash
claude mcp add puppeteer npx -y @anthropic-ai/mcp-puppeteer
```

### Design Tools

**Figma:**
```bash
claude mcp add figma npx -y @anthropic-ai/mcp-figma \
  --env FIGMA_ACCESS_TOKEN=xxx
```

### Productivity

**Linear:**
```bash
claude mcp add linear npx -y @anthropic-ai/mcp-linear \
  --env LINEAR_API_KEY=xxx
```

**Slack:**
```bash
claude mcp add slack npx -y @anthropic-ai/mcp-slack \
  --env SLACK_TOKEN=xxx
```

## Server Scopes

| Flag | Scope | Use Case |
|------|-------|----------|
| `--local` | Project only | Project-specific tools |
| `--global` | User-wide (default) | Personal tools |

```bash
# Project-specific
claude mcp add my-server --local npx my-package

# Global (default)
claude mcp add my-server npx my-package
```

## Configuration via JSON

### Location

```bash
~/.claude/settings.json       # User settings
.claude/settings.json         # Project settings
.claude/settings.local.json   # Local project settings
```

### Format

```json
{
  "mcpServers": {
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "postgres": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-postgres"],
      "args": ["postgresql://localhost/mydb"]
    }
  }
}
```

### Environment Variables

Use `${VAR_NAME}` syntax for secrets:

```json
{
  "env": {
    "API_KEY": "${MY_API_KEY}"
  }
}
```

## MCP Commands

| Command | Purpose |
|---------|---------|
| `claude mcp add <name> <cmd>` | Add server |
| `claude mcp remove <name>` | Remove server |
| `claude mcp list` | List all servers |
| `claude mcp get <name>` | Show server config |
| `claude mcp disable <name>` | Temporarily disable |
| `claude mcp enable <name>` | Re-enable server |

## Using MCP Resources

Reference MCP resources with `@`:

```
Show me the data from @github:repos/owner/repo/issues
```

Format: `@<server>:<resource-path>`

## MCP Permissions

Control MCP tool access in settings:

```json
{
  "permissions": {
    "allow": ["mcp__github__*"],
    "deny": ["mcp__postgres__execute_query"]
  }
}
```

Pattern: `mcp__<server>__<tool>`

### Auto-Approval

```json
{
  "mcpServers": {
    "myserver": {
      "autoApproveTools": true
    }
  }
}
```

## Troubleshooting

### Server Not Starting

1. Check command path: `which npx`
2. Verify package installed: `npx -y @package --help`
3. Check logs: `claude mcp get <name>`
4. Restart: `claude mcp remove <name> && claude mcp add ...`

### Server Health

```
/mcp
```

Shows connection status for all servers.

### Tool Not Found

```bash
# List available tools
claude mcp get <name>
```

### Authentication Issues

- Use environment variables for secrets
- Don't hardcode tokens in JSON
- Check token permissions/scopes

## Creating Custom MCP Servers

### TypeScript Template

```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new Server({
  name: "my-server",
  version: "1.0.0"
}, {
  capabilities: {
    tools: {}
  }
});

server.setRequestHandler("tools/list", async () => ({
  tools: [{
    name: "my_tool",
    description: "Does something useful",
    inputSchema: {
      type: "object",
      properties: {
        input: { type: "string" }
      }
    }
  }]
}));

server.setRequestHandler("tools/call", async (request) => {
  if (request.params.name === "my_tool") {
    return { content: [{ type: "text", text: "Result" }] };
  }
});

const transport = new StdioServerTransport();
await server.connect(transport);
```

### Python Template

```python
from mcp.server import Server
from mcp.server.stdio import stdio_server

server = Server("my-server")

@server.list_tools()
async def list_tools():
    return [{
        "name": "my_tool",
        "description": "Does something",
        "inputSchema": {"type": "object", "properties": {"input": {"type": "string"}}}
    }]

@server.call_tool()
async def call_tool(name, arguments):
    if name == "my_tool":
        return [{"type": "text", "text": "Result"}]

async def main():
    async with stdio_server() as streams:
        await server.run(*streams)

import asyncio
asyncio.run(main())
```

## Decision Guide

| Need | Solution |
|------|----------|
| GitHub access | MCP GitHub server |
| Database queries | MCP Postgres/SQLite |
| External APIs | MCP Fetch or custom server |
| Browser automation | MCP Puppeteer |
| Custom integrations | Build MCP server |

## References

- [MCP Server Registry](references/mcp-servers.md)
- [Building MCP Servers](references/building-mcp.md)
