# Google ADK (Agent Development Kit) Learning Notes

> Google ADK is an open-source, code-first Python framework for building, evaluating, and deploying AI agents — with native integration into Vertex AI, Gemini, and support for other models via LiteLLM.

> **Latest stable version: 1.32.0** (as of May 2026) | Also available in TypeScript, Go, and Java.

---

## Table of Contents

1. [What Is Google ADK?](#1-what-is-google-adk)
2. [Core Concepts](#2-core-concepts)
3. [Agent Types](#3-agent-types)
4. [Tools](#4-tools)
5. [Multi-Agent Orchestration](#5-multi-agent-orchestration)
6. [Sessions & Memory](#6-sessions--memory)
7. [Built-in Integrations](#7-built-in-integrations)
8. [Running & Deploying Agents](#8-running--deploying-agents)
9. [ADK vs. Other Frameworks](#9-adk-vs-other-frameworks)
10. [Production Best Practices](#10-production-best-practices)
11. [Tips & Mental Models](#11-tips--mental-models)

---

## 1. What Is Google ADK?

**Google ADK (Agent Development Kit)** is Google's open-source Python framework for building AI agents. It is **model-agnostic** (optimized for Gemini but works with Anthropic, OpenAI, and others via LiteLLM), **deployment-agnostic**, and **framework-compatible** (works alongside LangChain, MCP, etc.).

| | LangChain | CrewAI | Google ADK |
|---|---|---|---|
| Primary model | Any | Any | Any (optimized for Gemini) |
| Orchestration style | Chain-based | Role-based | Hierarchical / Event-driven |
| Multi-agent | Via LangGraph | First-class | First-class |
| Cloud integration | Manual | Manual | Native (Vertex AI) |
| Multi-language SDKs | Python only | Python only | Python, TypeScript, Go, Java |
| Open source | ✅ | ✅ | ✅ |

**When to use ADK:**
- You're building on Google Cloud / Vertex AI
- You want first-class Gemini integration with the option to swap models
- You need structured multi-agent workflows with Google tooling (Search, BigQuery, etc.)
- You want to deploy agents as scalable cloud services with minimal infra work

**Install:**
```bash
pip install google-adk

# With optional extensions (LiteLLM, etc.)
pip install "google-adk[extensions]"
```

---

## 2. Core Concepts

### The ADK Stack

```
┌────────────────────────────────────────────┐
│              Your Application              │
├────────────────────────────────────────────┤
│                  Runner                    │  ← Executes agent turns
├────────────────────────────────────────────┤
│                  Agent                     │  ← Defines behavior + tools
├──────────────┬─────────────────────────────┤
│    Model     │         Tools               │  ← LLM + callable functions
├──────────────┴─────────────────────────────┤
│          Session / Memory                  │  ← State across turns
└────────────────────────────────────────────┘
```

### Key Building Blocks

| Concept | What It Is |
|---|---|
| **Agent** | Core unit — combines a model, tools, instructions, and optional sub-agents |
| **Tool** | A Python function the agent can call to take action |
| **Runner** | Executes the agent loop turn by turn |
| **Session** | Conversation context within a single interaction |
| **Memory** | Persistent knowledge across sessions |
| **Artifact** | Binary output (files, images) produced during a run |
| **Event** | A unit in the agent's interaction log (messages, tool calls, tool results) |

---

## 3. Agent Types

ADK provides four main agent types. `Agent` is an alias for `LlmAgent` — both work.

### LlmAgent / Agent (Most Common)

The standard autonomous agent powered by a language model. The LLM decides which tools to call and when to stop.

```python
from google.adk.agents import Agent  # or LlmAgent — both are the same

agent = Agent(
    name="research_agent",
    model="gemini-2.5-flash",  # ← recommended latest model
    description="Researches topics using Google Search",
    instruction="You are a research assistant. Use search to find accurate, up-to-date information.",
    tools=[google_search],
)
```

**Use when:** The task requires dynamic reasoning — the agent needs to decide what to do next at each step.

---

### SequentialAgent (Workflow)

Runs a fixed list of sub-agents in order. Each agent's output is passed to the next.

```python
from google.adk.agents import SequentialAgent

pipeline = SequentialAgent(
    name="research_pipeline",
    sub_agents=[researcher, summarizer, formatter],
)
```

**Use when:** Task has a clear linear sequence — no branching or dynamic routing needed.

---

### ParallelAgent (Concurrent)

Runs multiple sub-agents simultaneously. Aggregates their outputs.

```python
from google.adk.agents import ParallelAgent

parallel_runner = ParallelAgent(
    name="multi_source_research",
    sub_agents=[news_agent, docs_agent, social_agent],
)
```

**Use when:** Sub-tasks are independent and can run at the same time for speed.

---

### LoopAgent (Iterative)

Runs a sub-agent repeatedly until a condition is met (or max iterations reached).

```python
from google.adk.agents import LoopAgent

refiner = LoopAgent(
    name="quality_loop",
    sub_agents=[draft_agent, review_agent],
    max_iterations=5,
)
```

**Use when:** You need an evaluator-optimizer pattern — generate, evaluate, refine, repeat.

---

### Agent Config (No-Code Option)

ADK also supports building agents via YAML/JSON config — no Python code required:

```yaml
# agent.yaml
name: my_agent
model: gemini-2.5-flash
instruction: "You are a helpful assistant."
tools:
  - google_search
```

**Use when:** You want to configure agents declaratively or manage them via config files.

---

## 4. Tools

Tools are Python functions the agent can call. ADK handles the schema generation automatically from type hints and docstrings.

### Defining a Custom Tool

```python
def get_stock_price(ticker: str) -> dict:
    """
    Fetches the current stock price for a given ticker symbol.

    Args:
        ticker: The stock ticker symbol (e.g., 'GOOG', 'AAPL')

    Returns:
        A dict with 'ticker' and 'price' keys.
    """
    return {"ticker": ticker, "price": 182.50}

agent = Agent(
    name="finance_agent",
    model="gemini-2.5-flash",
    tools=[get_stock_price],
)
```

**Key rule:** Write clear docstrings — the model uses them to decide when and how to call the tool.

---

### Built-in Tools

ADK ships with ready-to-use tools:

| Tool | What It Does |
|---|---|
| `google_search` | Web search via Google Search API |
| `built_in_code_execution` | Run Python code in a sandbox |
| `vertex_ai_search_tool` | Search your own Vertex AI Search data stores |
| `load_memory` | Retrieve from long-term memory |
| `exit_loop` | Signal a LoopAgent to stop iterating |

```python
from google.adk.tools import google_search, built_in_code_execution

agent = Agent(
    name="analyst",
    model="gemini-2.5-flash",
    tools=[google_search, built_in_code_execution],
)
```

---

### Tool Confirmation (Human-in-the-Loop)

ADK has a built-in **tool confirmation flow** — you can require human approval before a tool runs:

```python
from google.adk.tools.confirmation import requires_confirmation

@requires_confirmation
def delete_record(record_id: str) -> dict:
    """Permanently deletes a record. Requires confirmation."""
    # Will pause and ask user to confirm before executing
    return db.delete(record_id)
```

This is the recommended pattern for guarding irreversible or high-risk actions.

---

### Tool Context & State

Tools can read and write shared session state using `ToolContext`:

```python
from google.adk.tools import ToolContext

def save_result(data: str, tool_context: ToolContext) -> str:
    """Saves a result to shared session state."""
    tool_context.state["last_result"] = data
    return "Saved."
```

---

### OpenAPI / REST Tools

ADK can auto-generate tools from an OpenAPI spec — no manual wrapping needed:

```python
from google.adk.tools.openapi_tool import OpenAPIToolset

tools = OpenAPIToolset.from_spec("path/to/openapi.yaml")
agent = Agent(name="api_agent", tools=tools, ...)
```

---

### Long-Running Tools

For async operations, use `LongRunningFunctionTool`:

```python
from google.adk.tools import LongRunningFunctionTool

async def process_large_file(file_path: str) -> str:
    """Processes a large file asynchronously."""
    return "Processing complete"

tool = LongRunningFunctionTool(func=process_large_file)
```

---

## 5. Multi-Agent Orchestration

In ADK, you compose multi-agent systems using the `sub_agents` parameter directly — no extra wrapper needed.

### Hierarchical Agent Pattern

```
         Root Agent (Orchestrator)
              /           \
     Research Agent    Writing Agent
     (google_search)   (drafts content)
```

```python
from google.adk.agents import Agent

research_agent = Agent(
    name="researcher",
    model="gemini-2.5-flash",
    description="Searches the web for information on a given topic.",
    tools=[google_search],
)

writing_agent = Agent(
    name="writer",
    model="gemini-2.5-flash",
    description="Writes polished content based on provided research.",
)

# Simply pass sub_agents directly — no AgentTool wrapper needed
root_agent = Agent(
    name="coordinator",
    model="gemini-2.5-flash",
    description="Coordinates research and writing tasks.",
    sub_agents=[research_agent, writing_agent],  # ← direct assignment
)
```

The root agent automatically decides when to delegate to a sub-agent based on their `description` fields.

---

### A2A Protocol (Remote Agent-to-Agent)

For communicating with agents running on **different machines or services**, ADK integrates with the **A2A (Agent2Agent) Protocol** — an open standard for remote agent-to-agent communication:

```python
from google.adk.a2a import A2AClient

# Call a remote agent as if it were a local one
remote_agent = A2AClient(url="https://remote-agent-service.example.com")
```

**Use A2A when:** You're building distributed systems where agents run as separate services.

---

### Communication Between Agents

Agents share state through `session.state` (a shared key-value dict). Any agent or tool can read/write it:

```
Agent A → writes "research_data" to state
Agent B → reads "research_data" from state → processes it
```

This avoids passing large blobs through LLM context — store data in state, pass only references.

---

## 6. Sessions & Memory

### Session (Short-Term)

A session represents a single conversation. It holds the event history and current state.

```python
from google.adk.sessions import InMemorySessionService

session_service = InMemorySessionService()
session = session_service.create_session(
    app_name="my_app",
    user_id="user_123",
    session_id="session_abc",
)
```

**Session service options:**

| Service | Best For |
|---|---|
| `InMemorySessionService` | Development, testing |
| `VertexAiSessionService` | Production on GCP |
| `DatabaseSessionService` (Firestore) | Production with Firestore backend |

---

### Session Rewind

ADK supports **rewinding** a session to before a previous invocation — useful for debugging or letting users undo agent actions:

```python
runner.rewind(session_id=session.id, invocation_id=previous_invocation_id)
```

---

### Memory (Long-Term)

For state that persists beyond a single session, use a `MemoryService`:

```python
from google.adk.memory import InMemoryMemoryService

memory_service = InMemoryMemoryService()
memory_service.add_session_to_memory(session)
```

The built-in `load_memory` tool lets agents query long-term memory:

```python
from google.adk.tools import load_memory

agent = Agent(
    name="personal_assistant",
    tools=[load_memory],
    instruction="Use memory to recall past conversations and user preferences.",
)
```

**Memory service options:**

| Service | Best For |
|---|---|
| `InMemoryMemoryService` | Development, testing |
| `VertexAiRagMemoryService` | Production on GCP (uses Vertex AI RAG) |
| `VertexAiMemoryBankService` | Production with `memories.ingest_events` API |
| Custom | Your own vector DB (Pinecone, Weaviate, etc.) |

---

## 7. Built-in Integrations

### Supported Models

ADK is **model-agnostic**. It works with:

| Provider | Models | How |
|---|---|---|
| Google | Gemini 2.5 Flash/Pro, 1.5 Pro/Flash | Native |
| Anthropic | Claude 3.5/3.7 Sonnet, Opus | Via LiteLLM |
| OpenAI | GPT-4o, o3, etc. | Via LiteLLM |
| Others | Mistral, Cohere, etc. | Via LiteLLM |

**Recommended Gemini models (latest):**

| Model | Best For |
|---|---|
| `gemini-2.5-flash` | Default choice — fast, cost-effective, capable |
| `gemini-2.5-pro` | Complex reasoning, large context |
| `gemini-1.5-pro` | Long context (1M tokens); large document analysis |
| `gemini-1.5-flash` | Speed + efficiency balance |

```python
agent = Agent(
    name="my_agent",
    model="gemini-2.5-flash",  # ← current recommended default
    ...
)

# Using Anthropic via LiteLLM
agent = Agent(
    name="claude_agent",
    model="litellm/anthropic/claude-sonnet-4-5",
    ...
)
```

---

### Vertex AI (Google Cloud)

ADK deploys natively to Vertex AI Agent Engine — Google's managed runtime:

```python
import vertexai
from vertexai import agent_engines

vertexai.init(project="your-project", location="us-central1")

remote_app = agent_engines.create(
    agent_engine=your_agent,
    requirements=["google-adk"],
)
```

---

### MCP (Model Context Protocol)

ADK supports MCP servers as tool sources:

```python
from google.adk.tools.mcp_tool import MCPToolset, StdioServerParameters

mcp_tools = MCPToolset(
    connection_params=StdioServerParameters(
        command="npx",
        args=["-y", "@modelcontextprotocol/server-filesystem", "/tmp"],
    )
)

agent = Agent(
    name="file_agent",
    model="gemini-2.5-flash",
    tools=[mcp_tools],
)
```

---

### LangChain Tools

ADK can wrap existing LangChain tools:

```python
from google.adk.tools.langchain_tool import LangchainTool
from langchain_community.tools import WikipediaQueryRun

wikipedia_tool = LangchainTool(tool=WikipediaQueryRun(...))
agent = Agent(tools=[wikipedia_tool], ...)
```

---

## 8. Running & Deploying Agents

### Local Development — Runner

```python
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.genai import types

session_service = InMemorySessionService()
runner = Runner(
    agent=root_agent,
    app_name="my_app",
    session_service=session_service,
)

session = session_service.create_session(app_name="my_app", user_id="u1")
content = types.Content(role="user", parts=[types.Part(text="Research AI trends")])

for event in runner.run(user_id="u1", session_id=session.id, new_message=content):
    if event.is_final_response():
        print(event.content.parts[0].text)
```

---

### ADK CLI — Dev UI & Eval

```bash
# Start the built-in dev web UI (test, debug, inspect traces)
adk web

# Run in terminal
adk run path/to/your/agent

# Evaluate your agent against an eval set
adk eval \
    my_agent/ \
    my_agent/eval_set.evalset.json

# Deploy directly to Vertex AI Agent Engine
adk deploy agent_engine my_agent/

# Deploy to Cloud Run
adk deploy cloud_run my_agent/
```

---

### Deploy to Vertex AI Agent Engine

```python
import vertexai
from vertexai import agent_engines

vertexai.init(project="my-project", location="us-central1")

remote = agent_engines.create(
    agent_engine=AdkApp(agent=root_agent, enable_tracing=True),
    requirements=["google-adk>=1.0.0"],
    display_name="My Production Agent",
)
```

---

### Deploy as FastAPI / Cloud Run

```python
from google.adk.cli.fast_api import get_fast_api_app

app = get_fast_api_app(
    agent_dir="./my_agent",
    session_service_uri="agentengine://your-session-store",
    allow_origins=["https://your-frontend.com"],
)
```

Run with: `uvicorn main:app` — or `adk deploy cloud_run` to deploy to Cloud Run automatically.

---

### Observability — OpenTelemetry

ADK 1.32 ships with **native OpenTelemetry** tracing and agentic metrics — no extra setup needed:

```python
# Enable tracing
AdkApp(agent=root_agent, enable_tracing=True)
```

In production on GCP, traces appear automatically in **Cloud Trace** and metrics in **Cloud Monitoring**. Locally, any OTLP-compatible backend works (Jaeger, Grafana Tempo, etc.).

---

## 9. ADK vs. Other Frameworks

| Feature | Google ADK | LangChain | CrewAI | AutoGen |
|---|---|---|---|---|
| Primary model | Any (Gemini-optimized) | Any | Any | Any |
| Multi-agent | ✅ Native | ✅ Via LangGraph | ✅ Role-based | ✅ Conversation |
| Workflow agents | ✅ Sequential/Parallel/Loop | ✅ LangGraph | ❌ | ❌ |
| Built-in memory | ✅ | ✅ (complex) | Limited | Limited |
| MCP support | ✅ | ✅ | ❌ | ❌ |
| OpenAPI tools | ✅ | ✅ | ❌ | ❌ |
| A2A Protocol | ✅ | ❌ | ❌ | ❌ |
| Cloud deployment | ✅ Vertex AI + Cloud Run | Manual | Manual | Manual |
| Dev UI | ✅ Built-in `adk web` | ✅ LangSmith (paid) | ❌ | ❌ |
| Multi-language | Python, TS, Go, Java | Python | Python | Python |
| Learning curve | Medium | High | Low | Medium |

---

## 10. Production Best Practices

### Agent Design

- **One agent, one job** — keep each agent focused; use `sub_agents` for complex tasks
- **Clear instructions** — be explicit about what the agent should and should NOT do
- **Scope tools carefully** — give each agent only the tools it needs
- **Use tool confirmation** for irreversible actions — don't rely on prompts alone

```python
def safe_tool(query: str) -> dict:
    try:
        result = call_external_api(query)
        return {"status": "ok", "data": result}
    except Exception as e:
        return {"status": "error", "message": str(e)}
```

### State Management

- Use `session.state` for data shared within a session (not the LLM context)
- Use `MemoryService` for cross-session persistence
- Use `ArtifactService` for binary files — not session state
- Key naming convention: `"agent_name:key"` to avoid collisions in multi-agent setups
- Use **Firestore** `DatabaseSessionService` for production durability

### Cost & Latency

- Use `gemini-2.5-flash` by default — fast, cheap, capable
- Use parallel agents to reduce wall-clock time on independent sub-tasks
- Set `max_iterations` on LoopAgent — never let it run unbounded
- Enable tracing to spot expensive steps and optimize them

### Security

- Never put secrets in tool code — use Google Secret Manager or env vars
- Treat web search results as untrusted content (prompt injection risk)
- Add tool confirmation at the root agent level for high-risk actions
- Use IAM roles scoped to only the GCP resources each agent needs
- Keep ADK updated — security patches are released regularly (v1.32 includes CVE fixes)

---

## 11. Tips & Mental Models

### Mental Model

> An ADK agent is a **Gemini model (or any LLM) with a toolbelt and a team**. The model reads your instructions and decides which tool to pick or which sub-agent to delegate to. Your job is to give it the right tools, clear instructions, and the right teammates.

### The ADK Agent Loop

```
User message
    ↓
Runner.run()
    ↓
Agent reads: instructions + tools + session history + state
    ↓
LLM decides: call a tool OR delegate to sub-agent OR respond directly
    ↓
[If tool] → Execute (with optional confirmation) → Feed result back → loop
    ↓
[If done] → Final response event → return to user
```

### Do

- ✅ **Use `gemini-2.5-flash` by default** — current recommended model
- ✅ **Use `Agent` or `LlmAgent`** — they're the same class, pick whichever reads better
- ✅ **Use `sub_agents=[]` directly** — no AgentTool wrapper needed for sub-agents
- ✅ **Write detailed docstrings on tools** — the model reads them to decide how to use them
- ✅ **Use `session.state` to pass data** between agents — don't stuff large data into LLM context
- ✅ **Use tool confirmation for destructive actions** — built-in HITL is cleaner than prompts
- ✅ **Use `adk web` during development** — visual trace of every event, tool call, and state
- ✅ **Set `max_iterations` on LoopAgent** — always define a stopping condition
- ✅ **Use `adk deploy` CLI** — simplest path to Cloud Run or Vertex AI Agent Engine
- ✅ **Enable OpenTelemetry tracing in prod** — `AdkApp(agent=..., enable_tracing=True)`

### Avoid

- ❌ **Vague agent instructions** — the more specific, the better the behavior
- ❌ **Giant monolithic agents** — break complex workflows into specialized sub-agents
- ❌ **Ignoring tool errors** — always return error info; silent failures confuse the agent
- ❌ **Storing files in session state** — use ArtifactService for binary data
- ❌ **Skipping tracing in prod** — you can't debug what you can't observe
- ❌ **Over-trusting external content** — web results and user files can contain injections
- ❌ **Using `gemini-2.0-flash` as default** — `gemini-2.5-flash` is the current recommended version

### Quick Reference: Agent Type Cheat Sheet

| Situation | Use This Agent |
|---|---|
| Dynamic task, LLM decides next steps | `Agent` / `LlmAgent` |
| Fixed sequence of steps | `SequentialAgent` |
| Run multiple tasks at once | `ParallelAgent` |
| Generate → evaluate → refine loop | `LoopAgent` |
| Complex task needing specialization | `Agent` + `sub_agents=[...]` |
| Remote agent on another service | `A2AClient` |

---

## Reference: Official Resources

- [Google ADK GitHub (Python)](https://github.com/google/adk-python) — Source code and examples
- [ADK Documentation](https://google.github.io/adk-docs/) — Official docs
- [ADK Quickstart](https://google.github.io/adk-docs/get-started/quickstart/) — Build your first agent in 5 minutes
- [ADK Samples](https://github.com/google/adk-samples) — Real-world examples
- [ADK TypeScript](https://github.com/google/adk-js) — TypeScript/JS SDK
- [ADK Go](https://github.com/google/adk-go) — Go SDK
- [ADK Java](https://github.com/google/adk-java) — Java SDK
- [A2A Protocol](https://github.com/google-a2a/A2A/) — Agent-to-Agent communication spec
- [Vertex AI Agent Engine](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/overview) — Managed deployment
- [Gemini API Docs](https://ai.google.dev/gemini-api/docs) — Model reference
- [CHANGELOG](https://github.com/google/adk-python/blob/main/CHANGELOG.md) — Latest release notes

---

*Last updated: 2026-05-06 | ADK version: 1.32.0*
