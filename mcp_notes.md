# MCP (Model Context Protocol) Learning Notes

> MCP is the USB-C of AI agents — one open standard that connects any agent to any tool or data source, without custom integration code.

---

## Table of Contents

1. [What Is MCP and Why It Matters](#1-what-is-mcp-and-why-it-matters)
2. [Core Architecture](#2-core-architecture)
3. [Three Primitives: What MCP Exposes](#3-three-primitives-what-mcp-exposes)
4. [Transport: How They Connect](#4-transport-how-they-connect)
5. [Using Existing MCP Servers](#5-using-existing-mcp-servers)
6. [Building a Custom MCP Server](#6-building-a-custom-mcp-server)
7. [MCP in Agent AI Projects](#7-mcp-in-agent-ai-projects)
8. [Tips & Common Mistakes](#8-tips--common-mistakes)

---

## 1. What Is MCP and Why It Matters

**MCP (Model Context Protocol)** is an open standard created by Anthropic that defines how AI agents communicate with external tools and data sources.

### The Problem Before MCP

```
Without MCP — every team reinvents the wheel:

Agent A  →  custom GitHub code
Agent B  →  custom GitHub code
Agent C  →  custom GitHub code
           ... every agent, every tool, N×M integrations
```

### The Solution After MCP

```
With MCP — one standard, reusable across everything:

Any Agent  →  MCP Client  →  MCP Server (GitHub)  →  GitHub API
                         →  MCP Server (Postgres) →  Your DB
                         →  MCP Server (Slack)    →  Slack API
           write once, use everywhere
```

**Why it matters for you:**
- Use any of hundreds of pre-built MCP servers instead of writing integration code
- Your agent gains new capabilities by adding a config line — not a new codebase
- MCP is now supported by Claude Code, Cursor, VS Code Copilot, and most modern AI dev tools

---

## 2. Core Architecture

MCP has three roles. Every interaction involves all three.

```
┌─────────────────────────────────────────────────────────────┐
│  HOST (your app or AI tool)                                 │
│                                                             │
│  ┌─────────────────┐       ┌─────────────────┐             │
│  │   MCP Client    │       │   MCP Client    │  (one per   │
│  │  (for server A) │       │  (for server B) │   server)   │
│  └────────┬────────┘       └────────┬────────┘             │
└───────────┼─────────────────────────┼────────────────────┘
            │  MCP Protocol           │  MCP Protocol
            │  (JSON-RPC)             │  (JSON-RPC)
┌───────────▼──────────┐  ┌──────────▼──────────────────────┐
│   MCP Server A       │  │   MCP Server B                  │
│   (e.g. GitHub)      │  │   (e.g. PostgreSQL)             │
└──────────────────────┘  └─────────────────────────────────┘
```

| Role | What It Is | Example |
|---|---|---|
| **Host** | The app that embeds the AI agent | Claude Code, Cursor, your custom agent app |
| **MCP Client** | Lives inside the host; manages one server connection | Auto-created when you add an MCP server |
| **MCP Server** | Standalone process that exposes capabilities | `github-mcp-server`, `postgres-mcp-server` |

**Key rule:** one MCP Client per MCP Server. The Host manages multiple Clients simultaneously.

---

## 3. Three Primitives: What MCP Exposes

Every MCP server can expose up to three types of capabilities:

### Tools — Functions the Agent Can Call

The most important primitive. Tools let the agent *take actions*.

```
Tool: create_github_issue
Input:  { "repo": "myapp", "title": "Bug: login fails", "body": "..." }
Output: { "issue_number": 42, "url": "https://github.com/..." }
```

The LLM decides *when* to call a tool and with *what arguments*. The MCP server executes it and returns the result. The LLM reads the result and continues.

### Resources — Data the Agent Can Read

Named data sources the agent can access, like files or database rows.

```
Resource URI:   file:///project/README.md
Resource URI:   postgres://mydb/users/123
Resource URI:   github://repos/myapp/issues/42
```

Use resources when you want the agent to *read* something without triggering an action.

### Prompts — Pre-Built Prompt Templates

Reusable prompt templates exposed by the server, selectable by the user or agent.

```
Prompt: "code_review"
Arguments: { "language": "python", "style_guide": "pep8" }
→ Expands into a full review prompt with context injected
```

**Summary:**

| Primitive | Agent can... | Analogy |
|---|---|---|
| **Tools** | Call functions, trigger actions | API endpoints |
| **Resources** | Read data passively | GET requests / files |
| **Prompts** | Use pre-built prompt templates | Reusable system prompts |

---

## 4. Transport: How They Connect

MCP supports two transport mechanisms:

### stdio — For Local Servers

```
Host process
    │  spawns as subprocess
    ▼
MCP Server process
    │  communicates via stdin/stdout
    ▼
JSON-RPC messages
```

- The Host starts the server as a child process
- Messages flow over stdin/stdout (no network, no ports)
- **Best for**: local tools — filesystem, databases on the same machine, CLI tools

### HTTP + SSE — For Remote Servers

```
Host (MCP Client)
    │  HTTP POST (client → server requests)
    │  HTTP GET / SSE (server → client events)
    ▼
Remote MCP Server
    │
    ▼
External API / Service
```

- Server runs independently (cloud service, separate machine)
- Client sends requests via HTTP POST; server pushes events via SSE
- **Best for**: shared team tools, cloud services, SaaS integrations

**Which to use:**

| Use Case | Transport |
|---|---|
| Local DB, local files, your own laptop tools | stdio |
| Cloud-hosted tools, team-shared servers, SaaS | HTTP + SSE |
| Quick prototype | stdio (simpler) |
| Production multi-user agent | HTTP + SSE |

---

## 5. Using Existing MCP Servers

You rarely need to build one from scratch. Hundreds of pre-built MCP servers exist.

### Popular Servers

| Server | What It Gives Your Agent |
|---|---|
| `@modelcontextprotocol/server-github` | Read/write issues, PRs, code, comments |
| `@modelcontextprotocol/server-filesystem` | Read/write files outside the project |
| `@modelcontextprotocol/server-postgres` | Query your PostgreSQL database |
| `@modelcontextprotocol/server-brave-search` | Real-time web search |
| `@modelcontextprotocol/server-slack` | Read/post to Slack channels |
| `@modelcontextprotocol/server-memory` | Persistent key-value memory for the agent |

Find more at [MCP Servers Directory](https://modelcontextprotocol.io/servers).

### Configure in Claude Code

MCP config lives in `~/.claude/claude.json` (global) or `.mcp.json` (project-level).

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_your_token"
      }
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres",
               "postgresql://localhost/mydb"]
    }
  }
}
```

Or use the CLI:
```bash
claude mcp add github          # Interactive setup
claude mcp list                # See active servers
claude mcp remove github       # Remove
```

### Configure in a Custom Agent (Python)

```python
import anthropic

client = anthropic.Anthropic()

# Tell the agent what MCP server to connect to
response = client.beta.messages.create(
    model="claude-opus-4-7",
    max_tokens=1024,
    tools=[],                     # regular tools still work
    mcp_servers=[{
        "type": "url",            # HTTP+SSE transport
        "url": "https://my-mcp-server.com/mcp",
        "name": "my-tools"
    }],
    messages=[{"role": "user",
               "content": "Search for open bugs and summarize them"}]
)
```

---

## 6. Building a Custom MCP Server

When you need a tool that doesn't exist yet, building one is straightforward.

### Minimal Python MCP Server (~40 lines)

```python
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp import types

app = Server("my-tools")

# Register a tool
@app.list_tools()
async def list_tools() -> list[types.Tool]:
    return [
        types.Tool(
            name="get_weather",
            description="Get current weather for a city",
            inputSchema={
                "type": "object",
                "properties": {
                    "city": {"type": "string", "description": "City name"}
                },
                "required": ["city"]
            }
        )
    ]

# Implement the tool
@app.call_tool()
async def call_tool(name: str, arguments: dict) -> list[types.TextContent]:
    if name == "get_weather":
        city = arguments["city"]
        # your actual logic here
        weather = fetch_weather_api(city)
        return [types.TextContent(type="text", text=str(weather))]

# Run
if __name__ == "__main__":
    import asyncio
    asyncio.run(stdio_server(app))
```

Install the SDK: `pip install mcp`

### Minimal TypeScript MCP Server

```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new Server({ name: "my-tools", version: "1.0.0" });

server.setRequestHandler("tools/list", async () => ({
  tools: [{
    name: "get_weather",
    description: "Get weather for a city",
    inputSchema: {
      type: "object",
      properties: { city: { type: "string" } },
      required: ["city"]
    }
  }]
}));

server.setRequestHandler("tools/call", async (req) => {
  if (req.params.name === "get_weather") {
    const city = req.params.arguments.city;
    return { content: [{ type: "text", text: `Weather in ${city}: sunny` }] };
  }
});

const transport = new StdioServerTransport();
await server.connect(transport);
```

Install: `npm install @modelcontextprotocol/sdk`

---

## 7. MCP in Agent AI Projects

### The End-to-End Flow

```
User: "Find all open bugs assigned to me and create a Jira sprint for them"

Agent (LLM) decides:
  1. Call tool: github_list_issues(assignee="me", label="bug", state="open")
     ← MCP Client sends request to GitHub MCP Server
     ← MCP Server calls GitHub API, returns list of 5 issues

  2. LLM reads the 5 issues, extracts titles/descriptions

  3. Call tool: jira_create_sprint(name="Bug Fix Sprint", issues=[...])
     ← MCP Client sends request to Jira MCP Server
     ← MCP Server creates the sprint, returns sprint URL

  4. LLM formats a response: "Created sprint at [url] with 5 issues."
Done.
```

### Architecture Pattern: Agent + MCP

```
┌──────────────────────────────────────────────────┐
│  Your Agent Application                          │
│                                                  │
│  ┌──────────┐    ┌────────────────────────────┐  │
│  │   LLM    │◄──►│      Orchestrator          │  │
│  │ (Claude) │    │  (tool routing, loop ctrl) │  │
│  └──────────┘    └──────────────┬─────────────┘  │
│                                 │                 │
│              ┌──────────────────┼──────────────┐  │
│              ▼                  ▼              ▼  │
│         MCP Client         MCP Client     MCP Client
│              │                  │              │  │
└──────────────┼──────────────────┼──────────────┼──┘
               ▼                  ▼              ▼
          MCP Server         MCP Server     MCP Server
          (GitHub)           (Postgres)     (Custom)
```

### When to Use MCP vs. Direct Tool Calls

| Situation | Use |
|---|---|
| You want reusability across multiple agents | MCP |
| The tool will be shared across your team | MCP |
| You're using Claude Code or Cursor | MCP (native support) |
| Quick one-off tool in a single agent | Direct function/tool call |
| Prototyping something fast | Direct function call first, migrate to MCP later |

### Real Project Setup Checklist

```
1. [ ] Identify what external systems your agent needs to touch
2. [ ] Check if an MCP server already exists for each one
3. [ ] Add existing servers to .mcp.json with credentials
4. [ ] For custom needs: build a small MCP server (use Python SDK)
5. [ ] Test: ask your agent to use each tool; verify results
6. [ ] Commit .mcp.json to the repo (without secrets — use env vars)
```

---

## 8. Tips & Common Mistakes

### Do

- ✅ **Use `.mcp.json` at the project root** — commits with the repo, works in CI and team setups
- ✅ **Store secrets in env vars, not in the config file** — `"env": { "TOKEN": "${MY_TOKEN}" }`
- ✅ **Write tight tool descriptions** — the LLM uses the description to decide when and how to call the tool; vague descriptions = wrong calls
- ✅ **Return structured data from tools** — JSON is easier for the LLM to reason about than raw text
- ✅ **Start with stdio transport** — simpler to run locally, easier to debug

### Avoid

- ❌ **One mega-tool that does everything** — split by action; `search_issues` and `create_issue` beat `manage_issues(action="search")`
- ❌ **Putting credentials directly in `.mcp.json`** — use environment variables or a secrets manager
- ❌ **Skipping error messages** — if a tool fails silently, the LLM will hallucinate the result; always return a clear error string
- ❌ **Building custom MCP servers before checking what already exists** — check [modelcontextprotocol.io/servers](https://modelcontextprotocol.io/servers) first

### Debugging MCP

| Symptom | Likely Cause |
|---|---|
| Agent ignores available tools | Tool description is too vague; LLM doesn't know when to call it |
| Tool call returns empty/wrong data | Server not receiving the right args; check input schema |
| MCP server won't start | Missing env vars; wrong command in config |
| Agent calls the wrong tool | Overlapping tool names or descriptions; make them more distinct |

---

## The 20% That Gives You 80%

1. **MCP = USB-C for agents** — one standard, plug any tool into any agent without custom integration code
2. **Three primitives: Tools (do things), Resources (read things), Prompts (reuse templates)** — Tools are what you'll use 90% of the time
3. **stdio for local, HTTP+SSE for remote** — start with stdio; move to HTTP+SSE when you need remote/shared servers
4. **Use existing servers first** — check `modelcontextprotocol.io/servers` before writing a single line; building a custom server is only ~40 lines if you do need one
5. **Config lives in `.mcp.json`** — commit it to your repo (without secrets) and every agent and teammate gets the same tools automatically

---

*Last updated: 2026-04-25*
