# Agent AI Learning Notes

> An AI agent is an LLM given tools, memory, and a loop — so it can plan and act, not just reply.

---

## Table of Contents

1. [What Is an AI Agent?](#1-what-is-an-ai-agent)
2. [The Five Core Components](#2-the-five-core-components)
3. [MCP: The Tool Standard](#3-mcp-the-tool-standard)
4. [How It All Works Together](#4-how-it-all-works-together)
5. [Key Patterns](#5-key-patterns)
6. [Multi-Agent Systems](#6-multi-agent-systems)
7. [Tips & Mental Models](#7-tips--mental-models)

---

## 1. What Is an AI Agent?

A **chatbot** answers a question. An **agent** takes actions until a goal is achieved.

| | Chatbot | AI Agent |
|---|---|---|
| Input/Output | Text in → Text out | Goal in → Actions + Result out |
| Runs | Once per message | Multiple steps in a loop |
| Has tools | No | Yes (search, code, APIs, files…) |
| Has memory | Session only | Short + long-term |
| Plans | No | Yes |

**The minimal definition**: LLM + Tools + Loop.

Give a model tools and permission to keep going until the job is done — that's an agent.

---

## 2. The Five Core Components

```
┌─────────────────────────────────────────┐
│                 AGENT                   │
│                                         │
│  ┌─────────┐   ┌────────┐  ┌────────┐  │
│  │   LLM   │   │ Tools  │  │ Memory │  │
│  └────┬────┘   └────┬───┘  └───┬────┘  │
│       │             │          │        │
│  ┌────▼─────────────▼──────────▼────┐  │
│  │           Orchestrator            │  │
│  └────────────────┬──────────────────┘  │
│                   │                     │
│  ┌────────────────▼──────────────────┐  │
│  │             Planning              │  │
│  └───────────────────────────────────┘  │
└─────────────────────────────────────────┘
```

### 1. LLM (The Brain)
The model that reasons, decides what to do next, and generates outputs. Examples: Claude, GPT-4o, Gemini.

### 2. Tools (The Hands)
Functions the LLM can call to interact with the world.

| Tool Type | Examples |
|---|---|
| Code execution | Run Python, bash scripts |
| File system | Read, write, search files |
| Web | Search, fetch URLs |
| APIs | GitHub, Slack, databases |
| Other agents | Sub-agents as callable tools |

### 3. Memory (What It Knows)

| Type | What | Where |
|---|---|---|
| **In-context** | Current conversation, tool results | LLM context window |
| **External short-term** | Scratch pad for the current task | Vector DB / key-value store |
| **Long-term** | User preferences, past decisions | Persistent DB, files |

### 4. Planning (How It Thinks)
The agent breaks a goal into steps before acting. Three common strategies:

- **Chain-of-Thought**: Think step-by-step before answering
- **ReAct**: Reason → Act → Observe → repeat
- **Plan-and-Execute**: Write a full plan first, then run it step by step

### 5. Orchestrator (The Loop)
The control layer that drives the agent loop:

```
while goal_not_reached:
    thought = llm.think(goal, memory, tools)
    action  = llm.choose_tool(thought)
    result  = tool.run(action)
    memory.update(result)
```

The orchestrator decides when to stop, when to ask the human, and how to handle errors.

---

## 3. MCP: The Tool Standard

**MCP (Model Context Protocol)** is the open standard for connecting AI agents to external tools and data sources. Think of it as **USB-C for AI** — one standard plug, endless compatible devices.

### The Problem It Solves

Without MCP, every team hardcodes their own tool integrations. With MCP, tools are plug-and-play across any MCP-compatible agent.

```
Before MCP:                   After MCP:
Agent A → custom GitHub code  Agent (any) → MCP Client
Agent B → custom GitHub code        ↓
Agent C → custom GitHub code  MCP Server (GitHub)
... every team reinvents it         ↓
                               GitHub API
```

### Architecture

```
┌──────────────────┐      MCP Protocol      ┌─────────────────────┐
│   AI Agent       │ ◄──────────────────── ►│   MCP Server        │
│  (MCP Client)    │   JSON-RPC over stdio  │  (Tool Provider)    │
│                  │   or HTTP + SSE        │                     │
│  - Claude Code   │                        │  - GitHub MCP       │
│  - Cursor        │                        │  - PostgreSQL MCP   │
│  - Your custom   │                        │  - Filesystem MCP   │
│    agent         │                        │  - Your custom MCP  │
└──────────────────┘                        └─────────────────────┘
```

### What an MCP Server Exposes

| Primitive | Description | Example |
|---|---|---|
| **Tools** | Functions the LLM can call | `create_issue(title, body)` |
| **Resources** | Data sources the LLM can read | `file://project/README.md` |
| **Prompts** | Pre-built prompt templates | `code_review_template` |

### Real Example: GitHub MCP Server

```
Agent task: "Find all open bugs and create a summary issue"

1. Agent calls: list_issues(repo="myapp", label="bug", state="open")
   ← MCP Server fetches from GitHub API, returns JSON

2. Agent reads the issues, summarizes them

3. Agent calls: create_issue(title="Bug Summary", body="...")
   ← MCP Server posts to GitHub API

Done. Agent never needed hardcoded GitHub credentials or custom integration code.
```

### Key MCP Facts

- **Open standard** by Anthropic, now widely adopted (Cursor, VS Code, Claude Code, and more)
- MCP servers can be written in **any language** (Python, TypeScript, Go…) — a basic one is ~50 lines
- Transport: local servers use **stdio**; remote servers use **HTTP + SSE**
- Config lives in `claude.json` or `.mcp.json` — one config, all your tools

---

## 4. How It All Works Together

A complete agent handling a real task:

```
Goal: "Find all failing tests in CI and open GitHub issues for each one"

Step 1 — PLAN
  LLM breaks goal into: fetch CI results → parse failures → create issues

Step 2 — ACT (loop begins)
  LLM calls tool: get_ci_results(run_id="latest")
  MCP Server → GitHub API → returns JSON with 3 failures

Step 3 — OBSERVE
  LLM reads results, stores failures in memory

Step 4 — ACT
  LLM calls tool: create_issue(title="Test auth_test.py failing", ...)
  MCP Server → GitHub API → issue created

Step 5 — LOOP
  Repeat step 4 for remaining 2 failures

Step 6 — STOP
  LLM detects all issues created, goal achieved, returns summary
```

**The key insight**: The LLM doesn't just generate text — it generates *decisions* about what tool to call next, reads the result, and continues. The loop runs until done or the LLM decides to ask the human.

---

## 5. Key Patterns

### ReAct (Most Common)

```
Thought:  "I need to find which file has the bug"
Action:   search_code(query="NullPointerException")
Observation: Found in UserService.java line 42
Thought:  "I should read that file"
Action:   read_file("UserService.java")
Observation: [file contents]
Thought:  "The null check is missing, I'll fix it"
Action:   edit_file(...)
```
Each cycle: **think → act → observe**. Simple, transparent, debuggable.

### Tool Use (Structured Output)

Modern LLMs natively output structured tool calls:

```json
{
  "tool": "search_code",
  "parameters": { "query": "NullPointerException", "language": "java" }
}
```

The orchestrator routes this to the right tool, runs it, and feeds the result back to the LLM.

### Human-in-the-Loop

Agents can pause and ask for input when:
- Confidence is low
- An action is irreversible (e.g., deleting data)
- Ambiguity requires a human decision

```
Agent: "I found 3 matching records to delete. Proceed? (y/n)"
Human: "y"
Agent: [continues]
```

---

## 6. Multi-Agent Systems

For complex tasks, one agent isn't enough. You build a **network of specialized agents**.

```
         Orchestrator Agent
         (plans, delegates)
        /         |         \
       ▼          ▼          ▼
 Research     Code        Reviewer
  Agent       Agent        Agent
(web search) (writes code) (checks output)
```

### Key Roles

| Role | Responsibility |
|---|---|
| **Orchestrator** | Breaks down the goal, assigns to sub-agents, aggregates results |
| **Specialist** | Focused agent with specific tools (search, code, write, etc.) |
| **Critic/Reviewer** | Evaluates another agent's output for quality or correctness |

### When to Use Multi-Agent

- Task is too long for one context window
- Different subtasks need different tools/expertise
- You want parallel execution (multiple agents working simultaneously)
- You want a check on the main agent's work

---

## 7. Tips & Mental Models

### The Right Mental Model

> An agent is a **while loop with an LLM inside**. The LLM decides what to do each iteration. The loop ends when the LLM says it's done.

### Do

- ✅ **Give agents specific, measurable goals** — "summarize these 10 issues" beats "help me with GitHub"
- ✅ **Design tools to be small and composable** — one tool, one job
- ✅ **Add human checkpoints for irreversible actions** — deletes, sends, deploys
- ✅ **Use MCP for external integrations** — don't hardcode tool logic into your agent
- ✅ **Log every tool call and result** — essential for debugging agent loops

### Avoid

- ❌ **Giving agents vague goals** — they'll loop forever or hallucinate a path
- ❌ **Tools that do too much** — `do_everything()` is not a tool
- ❌ **No stop condition** — always define what "done" looks like
- ❌ **Skipping memory** — without memory, the agent re-reads everything every loop iteration

### Debugging an Agent

| Symptom | Likely Cause |
|---|---|
| Agent loops endlessly | No clear stop condition; goal too vague |
| Agent hallucinates tool results | Tool errors not fed back to LLM |
| Agent loses track of goal | Context window exceeded; add summarization |
| Agent takes wrong action | System prompt too vague; improve tool descriptions |

---

## The 20% That Gives You 80%

If you only remember four things:

1. **Agent = LLM + Tools + Loop** — the loop is what makes it agentic
2. **MCP = the plug-and-play standard** for connecting agents to tools — use existing servers before building your own
3. **ReAct pattern** — Think → Act → Observe is the fundamental loop inside almost every agent
4. **Design for the stop condition first** — know what "done" looks like before you start

---

*Last updated: 2026-04-25*
