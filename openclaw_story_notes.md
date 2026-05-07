# OpenClaw: The Story Behind It

> A retired iOS developer and a game engine creator built a space lobster that lives in your pocket. 🦞

---

## What Is OpenClaw?

A **self-hosted AI assistant** you can message from WhatsApp, Telegram, Discord, or iMessage — all running on YOUR computer with YOUR API keys.

```
Your phone (WhatsApp/Telegram) → Gateway (your computer) → AI (Claude/Gemini/GPT)
```

- **Language:** TypeScript / Node.js
- **License:** MIT (free, open source)
- **Stars:** 369k+ on GitHub
- **Created:** November 2025

---

## Who Built It?

### Peter Steinberger (@steipete)
- Founded **PSPDFKit** — a massive PDF SDK company used by Dropbox, Autodesk, etc.
- Retired, then came back because AI was too exciting
- Bio: *"Came back from retirement to mess with AI"*
- 49k+ GitHub followers

### Mario Zechner (@badlogic)
- Created **libGDX** — one of the most popular open-source game frameworks
- Built **Pi** — the coding agent inside OpenClaw
- Deep systems programming + security background

### Clawd 🦞
- The space lobster mascot. OpenClaw = CLAW + open source.

---

## Why Was It Built?

Peter wanted to **text his AI like texting a friend** — from WhatsApp, with full access to his files, tools, and calendar, running on his own machine.

**Problems it solves:**
| Before OpenClaw | After OpenClaw |
|---|---|
| AI stuck in browser tabs | AI in your WhatsApp/Telegram |
| Close tab = lose context | Persistent memory across sessions |
| Locked to one AI provider | Use Claude, Gemini, GPT — your choice |
| AI can't access your files | Agent reads files, sends emails, runs code |
| Data goes to corporate servers | Everything runs locally, you own your data |

---

## How They Thought About This (Design Story)

The interesting part of OpenClaw isn't the code — it's the *frame shift*. Most "AI assistant" products try to be **the chat app**. OpenClaw asked the opposite question:

> "What if the chat app is fixed (it's WhatsApp, that's not changing) — and the AI comes to *me* there?"

That single inversion is what generated almost every other design choice.

### The five questions Peter & Mario kept asking

1. **Why does my AI live in a browser tab?** — Because every vendor wants you locked to their UI. But the chat app you actually use is WhatsApp. So: bring the AI to the user, not the user to the AI.
2. **Why does it forget me every session?** — Because vendors don't want to give you portable memory. Memory is the moat. So: give users *their own* memory, in plain files they own.
3. **Why am I locked to one model?** — Because vendors bundle UI + model. If we strip the UI down to "a messaging gateway," the model becomes swappable.
4. **Why does my data leave my machine?** — Because someone else's server is the path of least resistance. But laptops are fast now, and the user already owns the API key. So: run it locally.
5. **Why is the agent locked to one set of tools?** — Because tools are usually compiled in. So: make tools *folders of markdown* that anyone can write, share, and remix. → Skills.

Each "why" pointed at a vendor incentive, and the answer was: **invert the default**.

### The pivotal design decisions (and the "instead-of" they replace)

| Decision | Instead of… | Why |
|---|---|---|
| **Chat app = front door** | Building a custom chat UI | The user already has WhatsApp open. Don't fight habit. |
| **Files as memory** | A vector DB or proprietary store | Files are inspectable, portable, diff-able, version-controllable. The user owns them. |
| **Markdown as the agent's "OS"** | Code-defined personality | Anyone can edit `SOUL.md` or `IDENTITY.md` — no rebuild, no deploy. |
| **Skills = folders, not plugins** | A plugin SDK with versioning | A folder with a SKILL.md is git-cloneable. Sharing is `cp -r`. |
| **Gateway, not a single bot** | One agent per chat app | Decouple "where the message comes from" from "who answers it." Add Discord = one adapter, not a rewrite. |
| **Bring-your-own-key** | Hosted with vendor keys | No subscription, no rate-limit games, no data leaving your machine. |
| **TypeScript core** | Python (the AI default) | The hot path is WebSocket fan-in/fan-out from chat networks. JS won that ecosystem. |
| **MIT license, no SaaS** | Freemium / "open core" | The product *is* the gateway. Monetizing it would re-create the lock-in they were escaping. |

### The architecture as a thesis

```
Identity is portable    →  IDENTITY.md, SOUL.md, USER.md   (files, not DB rows)
Memory is portable      →  MEMORY.md + memory/*.md         (files, not embeddings)
Skills are portable     →  skills/*/SKILL.md               (folders, not plugins)
Models are portable     →  model adapter layer             (no model lock-in)
Transports are portable →  WhatsApp/Telegram/Discord/iMessage adapters
```

The whole system is: **make every layer portable, and the user owns the stack.** When all five layers are portable, no single vendor can become the moat.

---

## How Was It Built?

### The Key Insight: Gateway Pattern

One process connects ALL your messaging apps to ALL your AI models:

```
WhatsApp ──┐                    ┌── Claude
Telegram ──┤── Gateway (Node) ──┤── Gemini
Discord  ──┤                    ├── GPT
iMessage ──┘                    └── Pi (coding agent)
```

### Why TypeScript (Not Python)?

- Chat apps = real-time WebSocket connections → Node.js is perfect for this
- Libraries for WhatsApp/Telegram/Discord already existed in JS
- TypeScript catches routing bugs at compile time
- Python would've been easier for AI/ML, but OpenClaw is a **messaging gateway** first

### How a Message Flows

```
1. You send "What's the weather?" on WhatsApp
2. Gateway receives it via WhatsApp Web protocol
3. Routes it to the right AI agent + session
4. AI calls weather tool → gets result
5. AI writes response → Gateway sends it back
6. You see the reply on WhatsApp ✅
```

---

## How the Agent Remembers You

Simple but effective — **files are memory**:

```
~/.openclaw/workspace/
  MEMORY.md        ← long-term curated memory
  SOUL.md          ← agent's personality
  USER.md          ← info about you
  memory/
    2026-05-06.md  ← today's notes
```

Each session, the agent reads these files to know who it is and who you are. No database needed.

---

## The Skill System

Skills = modular capabilities as folders:

```
skills/
  weather/SKILL.md     ← AI reads instructions, calls scripts
  github/SKILL.md
  gog/SKILL.md         ← Gmail, Calendar, Drive
```

The AI reads markdown instructions to learn how to use each tool. Community shares skills via **ClawHub**.

---

## How To Develop Something Like This (The Build Order)

If you wanted to build OpenClaw from scratch — or anything in this shape — Peter & Mario's path suggests an order. It is **not** the order most teams would try.

### The wrong order (what most teams do)

1. Pick a chat app to support
2. Spend 3 months building the perfect UI in it
3. Bolt on tools, memory, multi-model later
4. Realize you've shipped a vendor lock-in product

### The right order (what OpenClaw actually did)

**Stage 1 — Prove the gateway pattern (one transport, one model)**
- Pick *one* chat app you personally use (WhatsApp).
- Get a single message round-trip working: phone → your laptop → Claude → reply back.
- No memory, no tools, no skills yet. Just the loop.
- Goal: *can I text my AI from outside my browser?*

**Stage 2 — Add files-as-memory**
- Create `MEMORY.md` and start writing notes to it from inside the loop.
- Reload it on every turn. That's the whole memory system.
- Goal: *does my AI remember me across sessions without a database?*

**Stage 3 — Add the persona layer**
- Add `IDENTITY.md`, `SOUL.md`, `USER.md` — markdown read at session start.
- Now the same agent has a stable personality, even though the loop is stateless.
- Goal: *does my AI feel like the same entity each time?*

**Stage 4 — Decouple tools as skills**
- Move every capability into `skills/<name>/SKILL.md` + scripts.
- The agent reads the SKILL.md to learn how to use it — same way a human reads a README.
- Goal: *can I add a new capability by dropping a folder, with no code changes?*

**Stage 5 — Make the model swappable**
- Behind a thin adapter, route the same loop to Gemini or GPT.
- Skills don't change; identity doesn't change; only the LLM endpoint does.
- Goal: *did I just neutralize the model vendor?*

**Stage 6 — Add more transports (Telegram, Discord, iMessage)**
- Each one is a small adapter that emits the same internal message format.
- The gateway pattern from Stage 1 pays off here — you're writing adapters, not rewriting the agent.
- Goal: *does my AI follow me to whichever app I'm using?*

**Stage 7 — Open the ecosystem (ClawHub)**
- Skills are folders. Folders are git-cloneable. Make a registry.
- Now the community can extend the agent without touching core.
- Goal: *does the system grow without me growing the codebase?*

### Principles to copy if you're building in this space

- **Inversion is free leverage.** Every default vendors set ("AI in our app", "memory in our DB", "tools in our store") is a candidate for inversion. The inverted version is often the user-respecting product.
- **Build the smallest end-to-end loop first.** A working WhatsApp ↔ Claude pipe with no memory is more valuable than a perfect memory system with no transport.
- **Choose data formats users can read.** Markdown over JSON, files over DB rows, folders over packages. If users can `cat` and `vim` it, they trust it.
- **Decouple along the axis of vendor power.** Anywhere a vendor could lock you in (transport, model, storage), put an adapter.
- **Ship adoption-as-a-format, not adoption-as-an-API.** A SKILL.md that's `git clone`-able will be remixed 10× more than an SDK that requires `npm install`.
- **Personality is a real feature.** A space lobster mascot that calls you by name is not branding — it's *retention*. Friction-free identity is what makes people text the bot daily.

---

## How It Grew

| Date | Milestone |
|---|---|
| Nov 2025 | First release |
| Early 2026 | WhatsApp, Telegram, Discord, iMessage all working |
| Mid 2026 | 369k+ stars, iOS/Android nodes, web dashboard, plugin ecosystem |

For context: React has 233k stars. OpenClaw passed it because it solved a daily pain point for millions of people.

---

## The Philosophy

1. **Own your data** — runs on your hardware, no corporate middleman
2. **Any OS, any platform** — Mac, Linux, Windows, Raspberry Pi
3. **Agent-native** — built for AI agents with tools, memory, and sessions (not just a chatbot)
4. **Fun over corporate** — space lobster > sterile enterprise branding 🦞

---

## One-Liner Summary

> Peter built what he wished existed: an AI you can WhatsApp from anywhere, that runs on your own machine, remembers you, and has the personality of a space lobster.

---

## Links

- [GitHub](https://github.com/openclaw/openclaw) | [Docs](https://docs.openclaw.ai) | [ClawHub](https://clawhub.com) | [Discord](https://discord.com/invite/clawd)
- Peter: [@steipete](https://x.com/steipete) | Mario: [@badlogicgames](https://x.com/badlogicgames)

---

*Last updated: 2026-05-06*
