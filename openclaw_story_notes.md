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
