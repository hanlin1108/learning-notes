# Vulnerability Scanning Agent — Learning Notes

> A practical walkthrough of building a single-agent code security scanner with **Google ADK + Gemini 2.5 Pro (Vertex AI)**. The agent autonomously decides which files to read, which analysis tools to invoke, and produces a structured vulnerability report — with every reasoning step printed to the terminal.

---

## Table of Contents

1. [What We Built](#1-what-we-built)
2. [Architecture at a Glance](#2-architecture-at-a-glance)
3. [The 6 Tools](#3-the-6-tools)
4. [The Agent Loop in ADK](#4-the-agent-loop-in-adk)
5. [Vertex AI Setup (ADC)](#5-vertex-ai-setup-adc)
6. [Observability Pattern](#6-observability-pattern)
7. [Key Takeaways](#7-key-takeaways)

---

## 1. What We Built

A CLI agent that scans a local codebase, finds security vulnerabilities, and writes a report.

```bash
python main.py --codebase ./demo_target
# → scan_results/report.json + scan_results/report.md
```

**Core demo value:** the agent's multi-round reasoning is **visible** — every thought, tool call, and result is streamed to stdout.

**Strict scope (MVP rules):**
- Single `LlmAgent` only — no multi-agent, no workflow agents, no sub-agents
- `gemini-2.5-pro` via **Vertex AI** (NOT AI Studio API key)
- Auth via **Application Default Credentials** (ADC)
- Max 15 LLM calls per run (hard cap)
- ~500 lines of code total

---

## 2. Architecture at a Glance

```
       ┌────────────────────────────────┐
       │   main.py  (CLI + observer)    │
       └──────────────┬─────────────────┘
                      │ Runner.run_async()
                      ▼
       ┌────────────────────────────────┐
       │   LlmAgent (Gemini 2.5 Pro)    │
       │   + system instruction         │
       └──────────────┬─────────────────┘
                      │ tool calls
                      ▼
   ┌──────────────────────────────────────────┐
   │  6 Python functions registered as tools  │
   │  list_codebase · read_file · run_sast    │
   │  call_mythos · search_known_vulns        │
   │  write_report                            │
   └──────────────────────────────────────────┘
```

**One key decision:** No orchestration logic in code. The LLM decides the order and stopping point. We just give it good tools and clear instructions.

---

## 3. The 6 Tools

| # | Tool | Real / Mock | Purpose |
|---|------|-------------|---------|
| 1 | `list_codebase(root_path)` | Real | Walk the dir, return `{path, size, language}` for each source file |
| 2 | `read_file(path, max_lines)` | Real | Read file content (capped at 500 lines) |
| 3 | `run_sast(path)` | Mock | Pretend SAST: flag `execute()` + string concat as SQL injection |
| 4 | `call_mythos(snippet, file)` | Mock | Pretend internal scanner: detect `eval()`, `password ==`, `shell=True` |
| 5 | `search_known_vulns(query)` | Mock | Pretend RAG: return historical incidents from a keyword dict |
| 6 | `write_report(findings, path)` | Real | Write `report.json` + grouped-by-severity `report.md` |

**Why the mocks?** Demonstrates the **cross-validation pattern** (SAST + internal scanner + historical lookup) without needing real infra. The agent learns to combine multiple weak signals into stronger conclusions.

**Tool registration is just Python:**

```python
from google.adk.agents import Agent

root_agent = Agent(
    name="vuln_scanner",
    model="gemini-2.5-pro",
    instruction=SYSTEM_INSTRUCTION,
    tools=[list_codebase, read_file, run_sast,
           call_mythos, search_known_vulns, write_report],
)
```

ADK auto-generates the JSON schema from type hints + docstrings. **Good docstrings = good tool selection** by the model.

---

## 4. The Agent Loop in ADK

ADK exposes the agent loop through `Runner.run_async()`, which **yields events** — one per LLM turn or tool call.

```python
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.genai.types import Content, Part

session_service = InMemorySessionService()
session = await session_service.create_session(
    app_name="vuln_scanner", user_id="cli_user"
)

runner = Runner(
    agent=root_agent,
    app_name="vuln_scanner",
    session_service=session_service,
)

async for event in runner.run_async(
    user_id="cli_user",
    session_id=session.id,
    new_message=Content(role="user", parts=[Part(text=prompt)]),
):
    # inspect event.content.parts → text / function_call / function_response
    ...
```

Each `event` carries `content.parts`, where a part is one of:

| Part field | Meaning |
|------------|---------|
| `text` | LLM's reasoning or final answer |
| `function_call` | The model wants to call a tool — name + args |
| `function_response` | Tool returned a result back to the model |

**This event stream is the entire observability surface.** Iterate it, pretty-print it, you're done.

---

## 5. Vertex AI Setup (ADC)

The non-obvious part. **Three things must be true** or the agent won't start.

### 5.1 Authenticate with Application Default Credentials

```bash
gcloud auth application-default login
```

Opens a browser. Saves credentials to `~/.config/gcloud/application_default_credentials.json`. The Google GenAI SDK picks them up automatically.

### 5.2 Set the three required env vars

```bash
GOOGLE_GENAI_USE_VERTEXAI=true       # routes SDK calls to Vertex, not AI Studio
GOOGLE_CLOUD_PROJECT=your-project-id
GOOGLE_CLOUD_LOCATION=us-central1
```

### 5.3 Do NOT set API keys

```bash
# Wrong — will silently override Vertex AI routing
GOOGLE_API_KEY=...
GEMINI_API_KEY=...
```

> **Trap:** if either API key var is set, the SDK takes the AI Studio path and ignores `GOOGLE_GENAI_USE_VERTEXAI`. Unset them.

### 5.4 Enable the Vertex AI API once per project

```bash
gcloud services enable aiplatform.googleapis.com
```

---

## 6. Observability Pattern

The selling point of the demo. Pattern:

```python
async for event in runner.run_async(...):
    if event.content and event.content.parts:
        for part in event.content.parts:
            if part.text:
                round_num += 1
                print(f"[Round {round_num}] 🧠 Thinking: {part.text[:200]}")

            if part.function_call:
                fc = part.function_call
                print(f"[Tool] {fc.name}({dict(fc.args)})", end=" → ")

            if part.function_response:
                print(_summarise(part.function_response.response))
```

**Resulting trace:**

```
[Round 1] 🧠 Thinking: I'll start by listing the codebase structure...
[Tool] list_codebase({'root_path': '...'}) → [{'path': 'auth.py', ...}, ...]

[Round 2] 🧠 Thinking: auth.py looks risky (password handling)...
[Tool] read_file({'path': '.../auth.py'}) → def login(...): if password == ...
[Tool] call_mythos({...}) → {'vulnerabilities': [{'cwe': 'CWE-208', ...}]}
[Tool] search_known_vulns({...}) → [{'incident_id': 'INC-2022-0119', ...}]
```

**Why this matters:** when you can *see* the model's plan, debugging tool design and instruction tuning becomes trivial. Bad output? You can pinpoint which round and which tool result confused it.

---

## 7. Key Takeaways

### Do
- ✅ **Let the LLM drive** — don't hard-code the scan order; give it tools and trust the instruction
- ✅ **Write detailed docstrings on every tool** — that's the model's API documentation
- ✅ **Cap LLM calls** with a counter and a `MAX_LLM_CALLS` constant — protects against infinite loops
- ✅ **Stream events** for observability — `async for event in runner.run_async()`
- ✅ **Validate env vars at startup** — fail fast with a clear message if Vertex AI isn't configured
- ✅ **Use mocks for downstream tools** when prototyping — the agent's *behavior* is what's interesting, not the tool internals

### Avoid
- ❌ **Multi-agent setups for an MVP** — one well-instructed agent with good tools beats premature orchestration
- ❌ **API keys for Vertex AI** — `GOOGLE_API_KEY` silently breaks the ADC path
- ❌ **Letting the agent loop unbounded** — always cap rounds (here: 15)
- ❌ **Hiding the reasoning** — printing the trace is what makes the demo land
- ❌ **Big monolithic tools** — many small focused tools (read_file, run_sast, call_mythos, …) compose better than one mega-tool

### Mental Model

> The agent is a **planner with a cheat sheet** (the system instruction) and a **toolbox** (the 6 functions). Your job is to make the cheat sheet specific and the toolbox sharp. The Runner just turns the crank and emits events you can watch.

---

## Reference: Project Layout

```
vulner_scanning_agent_demo/
├── requirements.txt
├── .env.example
├── README.md
└── vuln_agent/
    ├── agent.py        # Agent definition + system instruction
    ├── tools.py        # The 6 tool functions
    ├── main.py         # CLI + event-stream observer
    └── demo_target/    # Planted vulns: CWE-208, CWE-89, CWE-95, CWE-78
```

**Run it:**

```bash
cd vuln_agent
python main.py --codebase ./demo_target
# → scan_results/report.json
# → scan_results/report.md
```

---

*Last updated: 2026-05-08*
