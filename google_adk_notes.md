# Google ADK (Agent Development Kit) Learning Notes

> Google ADK is an open-source Python framework for building, orchestrating, and deploying AI agents — with native integration into Vertex AI and Gemini models.

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

**Google ADK (Agent Development Kit)** is Google's open-source Python framework for building AI agents. It provides a structured way to define agents, connect them to tools, manage memory and sessions, and deploy to Google Cloud.

| | LangChain | CrewAI | Google ADK |
|---|---|---|---|
| Primary model | Any | Any | Gemini (Vertex AI) |
| Orchestration style | Chain-based | Role-based | Hierarchical / Event-driven |
| Multi-agent | Plugin | First-class | First-class |
| Cloud integration | Manual | Manual | Native (Vertex AI) |
| Open source | ✅ | ✅ | ✅ |

**When to use ADK:**
- You're building on Google Cloud / Vertex AI
- You want first-class Gemini integration
- You need structured multi-agent workflows with Google tooling (Search, BigQuery, etc.)
- You want to deploy agents as scalable cloud services

**Install:**
```bash
pip install google-adk
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

ADK provides three main agent types. Pick the one that fits the task.

### LlmAgent (Most Common)

The standard autonomous agent powered by a language model. The LLM decides which tools to call and when to stop.

```python
from google.adk.agents import LlmAgent
from google.adk.tools import google_search

agent = LlmAgent(
    name="research_agent",
    model="gemini-2.0-flash",
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
    # Your logic here
    return {"ticker": ticker, "price": 182.50}

agent = LlmAgent(
    name="finance_agent",
    model="gemini-2.0-flash",
    tools=[get_stock_price],
)
```

**Key rule:** Write clear docstrings — the model uses them to decide when and how to call the tool.

---

### Built-in Tools

ADK ships with ready-to-use tools for common Google services:

| Tool | What It Does |
|---|---|
| `google_search` | Web search via Google Search API |
| `built_in_code_execution` | Run Python code in a sandbox |
| `vertex_ai_search_tool` | Search your own Vertex AI Search data stores |
| `load_memory` | Retrieve from long-term memory |
| `exit_loop` | Signal a LoopAgent to stop iterating |

```python
from google.adk.tools import google_search, built_in_code_execution

agent = LlmAgent(
    name="analyst",
    model="gemini-2.0-flash",
    tools=[google_search, built_in_code_execution],
)
```

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

State stored in `tool_context.state` persists across tool calls within a session and is visible to all agents in a multi-agent setup.

---

### Long-Running Tools

For async operations (file processing, external API calls that take time), use `LongRunningFunctionTool`:

```python
from google.adk.tools import LongRunningFunctionTool

async def process_large_file(file_path: str) -> str:
    """Processes a large file asynchronously."""
    # async work here
    return "Processing complete"

tool = LongRunningFunctionTool(func=process_large_file)
```

---

## 5. Multi-Agent Orchestration

In ADK, agents can call other agents as tools. This is the foundation of multi-agent systems.

### Hierarchical Agent Pattern

```
         Root Agent (Orchestrator)
              /           \
     Research Agent    Writing Agent
     (google_search)   (drafts content)
```

```python
from google.adk.agents import LlmAgent
from google.adk.tools import agent_tool

research_agent = LlmAgent(
    name="researcher",
    model="gemini-2.0-flash",
    description="Searches the web for information on a given topic.",
    tools=[google_search],
)

writing_agent = LlmAgent(
    name="writer",
    model="gemini-2.0-flash",
    description="Writes polished content based on provided research.",
)

root_agent = LlmAgent(
    name="coordinator",
    model="gemini-2.0-flash",
    description="Coordinates research and writing tasks.",
    tools=[
        agent_tool.AgentTool(agent=research_agent),
        agent_tool.AgentTool(agent=writing_agent),
    ],
)
```

The root agent decides when to delegate to a sub-agent, just like it decides when to call any other tool.

---

### Communication Between Agents

Agents share state through `session.state` (a shared key-value dict). Any agent or tool can read/write it:

```
Agent A → writes "research_data" to state
Agent B → reads "research_data" from state → processes it
```

This avoids passing large blobs through LLM context — store data in state, pass only references.

---

### Trust & Safety in Multi-Agent

- **Orchestrators** are trusted (they're your code)
- **Sub-agents** should be scoped — give them only the tools they need
- **External content** (web results, user uploads) is untrusted — handle carefully
- Add guardrails at the root agent level for high-risk actions

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

Session state is a dict you can use to pass data between agents and tools:

```python
session.state["user_preference"] = "concise answers"
```

---

### Memory (Long-Term)

For state that persists beyond a single session, use a `MemoryService`:

```python
from google.adk.memory import InMemoryMemoryService

memory_service = InMemoryMemoryService()
# Ingest a completed session into long-term memory
memory_service.add_session_to_memory(session)
```

The built-in `load_memory` tool lets agents query long-term memory during a conversation:

```python
from google.adk.tools import load_memory

agent = LlmAgent(
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
| Custom | Your own vector DB (Pinecone, Weaviate, etc.) |

---

### Artifact Storage

Agents can produce file outputs (images, PDFs, CSVs). These are stored as artifacts:

```python
from google.adk.artifacts import InMemoryArtifactService

artifact_service = InMemoryArtifactService()
```

Artifacts are separate from session state — they're for binary blobs, not structured data.

---

## 7. Built-in Integrations

### Vertex AI (Google Cloud)

ADK is built to run on Vertex AI Agent Engine — Google's managed runtime for agents.

```python
import vertexai
from vertexai import agent_engines

vertexai.init(project="your-project", location="us-central1")

# Deploy your agent
remote_app = agent_engines.create(
    agent_engine=your_agent,
    requirements=["google-adk"],
)
```

Once deployed, your agent runs as a scalable cloud service with built-in logging and monitoring.

---

### Gemini Models

ADK natively supports all Gemini models:

| Model | Best For |
|---|---|
| `gemini-2.0-flash` | Fast, cost-effective; great for most agents |
| `gemini-2.0-flash-thinking` | Complex reasoning with visible thought process |
| `gemini-1.5-pro` | Long context (1M tokens); large document analysis |
| `gemini-1.5-flash` | Speed + efficiency balance |

```python
agent = LlmAgent(
    name="my_agent",
    model="gemini-2.0-flash",  # or "gemini-1.5-pro" etc.
    ...
)
```

---

### MCP (Model Context Protocol)

ADK supports MCP servers as tool sources — plug in any MCP-compatible server:

```python
from google.adk.tools.mcp_tool import MCPToolset, StdioServerParameters

mcp_tools = MCPToolset(
    connection_params=StdioServerParameters(
        command="npx",
        args=["-y", "@modelcontextprotocol/server-filesystem", "/tmp"],
    )
)

agent = LlmAgent(
    name="file_agent",
    model="gemini-2.0-flash",
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

agent = LlmAgent(tools=[wikipedia_tool], ...)
```

---

## 8. Running & Deploying Agents

### Local Development — Runner

The `Runner` executes the agent turn by turn:

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

# Run a turn
session = session_service.create_session(app_name="my_app", user_id="u1")
content = types.Content(role="user", parts=[types.Part(text="Research AI trends")])

for event in runner.run(user_id="u1", session_id=session.id, new_message=content):
    if event.is_final_response():
        print(event.content.parts[0].text)
```

---

### ADK CLI (Dev Web UI)

ADK ships with a built-in dev server and web UI for testing:

```bash
# Start the dev UI
adk web

# Or run in terminal
adk run path/to/your/agent
```

The web UI shows:
- The conversation
- Which tools were called
- The full event trace (great for debugging)
- State and artifact inspection

---

### Deploy to Vertex AI Agent Engine

```python
import vertexai
from vertexai import agent_engines

vertexai.init(project="my-project", location="us-central1")

# Create the remote agent
remote = agent_engines.create(
    agent_engine=AdkApp(agent=root_agent, enable_tracing=True),
    requirements=["google-adk>=0.5.0"],
    display_name="My Production Agent",
)

print(remote.resource_name)  # projects/.../reasoningEngines/...
```

Once deployed, call it via the Vertex AI SDK from anywhere in your app.

---

### Deploy as API (FastAPI)

For custom APIs, ADK generates a FastAPI app from your agent:

```python
from google.adk.cli.fast_api import get_fast_api_app

app = get_fast_api_app(
    agent_dir="./my_agent",
    session_service_uri="agentengine://your-session-store",
    allow_origins=["https://your-frontend.com"],
)
```

Run with: `uvicorn main:app`

---

## 9. ADK vs. Other Frameworks

| Feature | Google ADK | LangChain | CrewAI | AutoGen |
|---|---|---|---|---|
| Primary model | Gemini | Any | Any | Any |
| Multi-agent | ✅ Native | ✅ Via LangGraph | ✅ Role-based | ✅ Conversation-based |
| Workflow agents | ✅ Sequential/Parallel/Loop | ✅ Via LangGraph | ❌ | ❌ |
| Built-in memory | ✅ | ✅ (complex) | Limited | Limited |
| MCP support | ✅ | ✅ | ❌ | ❌ |
| Cloud deployment | ✅ Vertex AI native | Manual | Manual | Manual |
| Dev UI | ✅ Built-in | ✅ LangSmith (paid) | ❌ | ❌ |
| Learning curve | Medium | High | Low | Medium |

**Bottom line:**
- Use **ADK** if you're on GCP + Gemini and want managed deployment
- Use **LangGraph** for maximum flexibility with any LLM
- Use **CrewAI** for quick role-based agents with minimal boilerplate
- Use **AutoGen** for conversational multi-agent research setups

---

## 10. Production Best Practices

### Agent Design

- **One agent, one job** — keep each agent focused; use multi-agent for complex tasks
- **Clear instructions** — be explicit about what the agent should and should NOT do
- **Scope tools carefully** — give each agent only the tools it needs for its role
- **Handle tool errors** — always return error info from tools so the agent can adapt

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
- Don't store large blobs in state — use `ArtifactService` for files
- Key naming convention: `"agent_name:key"` to avoid collisions in multi-agent setups

### Cost & Latency

- Use `gemini-2.0-flash` by default — it's 10x cheaper than Pro for most tasks
- Use parallel agents to reduce wall-clock time on independent sub-tasks
- Set `max_iterations` on LoopAgent — never let it run unbounded
- Cache repeated context where possible (system instructions don't change between turns)

### Observability

- Enable tracing when deploying: `AdkApp(agent=..., enable_tracing=True)`
- Use the ADK dev UI (`adk web`) during development — inspect every tool call and event
- In production, Cloud Trace + Cloud Logging capture the full agent trace on Vertex AI
- Log `session.state` at the end of each run for debugging

### Security

- Never put secrets in tool code — use Secret Manager or env vars
- Treat web search results as untrusted content (prompt injection risk)
- Add a guardrails layer at the root agent for high-risk actions (deletes, payments, sends)
- Use IAM roles scoped to only the GCP resources each agent needs

---

## 11. Tips & Mental Models

### Mental Model

> An ADK agent is a **Gemini model with a toolbelt**. The model reads your instructions and decides which tool to pick. Your job as a developer is to give it the right tools and clear instructions about when to use them.

### The ADK Agent Loop

```
User message
    ↓
Runner.run()
    ↓
Agent reads: instructions + tools + session history + state
    ↓
LLM decides: call a tool OR respond directly
    ↓
[If tool] → Execute tool → Feed result back → loop
    ↓
[If done] → Final response event → return to user
```

### Do

- ✅ **Use `gemini-2.0-flash` by default** — fast, cheap, capable
- ✅ **Write detailed docstrings on tools** — the model reads them to decide how to use the tool
- ✅ **Use `session.state` to pass data** between agents — don't stuff large data into LLM context
- ✅ **Use SequentialAgent for pipelines** — deterministic, predictable, easy to debug
- ✅ **Use `adk web` during development** — visual trace of every event, tool call, and state change
- ✅ **Set `max_iterations` on LoopAgent** — always define a stopping condition
- ✅ **Return structured dicts from tools** — easier for the LLM to parse and reason about
- ✅ **Deploy to Vertex AI Agent Engine** — managed scaling, logging, and tracing out of the box

### Avoid

- ❌ **Vague agent instructions** — the more specific, the better the behavior
- ❌ **Giant monolithic agents** — break complex workflows into specialized sub-agents
- ❌ **Ignoring tool errors** — always return error info; silent failures confuse the agent
- ❌ **Storing files in session state** — use ArtifactService for binary data
- ❌ **Skipping tracing in prod** — you can't debug what you can't observe
- ❌ **Over-trusting external content** — web results and user files can contain injections

### Quick Reference: Agent Type Cheat Sheet

| Situation | Use This Agent |
|---|---|
| Dynamic task, LLM decides next steps | `LlmAgent` |
| Fixed sequence of steps | `SequentialAgent` |
| Run multiple tasks at once | `ParallelAgent` |
| Generate → evaluate → refine loop | `LoopAgent` |
| Complex task needing specialization | `LlmAgent` + sub-agents as tools |

---

## Reference: Official Resources

- [Google ADK GitHub](https://github.com/google/adk-python) — Source code and examples
- [ADK Documentation](https://google.github.io/adk-docs/) — Official docs
- [ADK Quickstart](https://google.github.io/adk-docs/get-started/quickstart/) — Build your first agent in 5 minutes
- [Vertex AI Agent Engine](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/overview) — Managed deployment
- [ADK Sample Agents](https://github.com/google/adk-samples) — Real-world examples
- [Gemini API Docs](https://ai.google.dev/gemini-api/docs) — Model reference

---

*Last updated: 2026-05-06*
