# OpenClaw — The Story 🦞

> A self-hosted AI assistant you text from WhatsApp/Telegram/Discord. Runs on YOUR computer, with YOUR API keys, and remembers you across conversations.

---

## The One-Liner

> Peter built what he wished existed: an AI you text like a friend, that lives on your laptop and uses your own model API keys.

---

## Who Built It?

- **Peter Steinberger** ([@steipete](https://x.com/steipete)) — founded PSPDFKit, came back from retirement for AI
- **Mario Zechner** ([@badlogicgames](https://x.com/badlogicgames)) — created libGDX game framework, built **Pi** (the coding agent inside)
- **Clawd** 🦞 — the space lobster mascot

Created November 2025 · MIT license · TypeScript/Node.js · 369k+ GitHub stars

---

## The Big Idea (Why It Exists)

Most AI assistants make YOU come to THEIR app. OpenClaw flips it:

> "WhatsApp isn't going anywhere — bring the AI to me there."

That single inversion drives everything else.

### What problems it solves

| Before | After |
|---|---|
| AI lives in a browser tab | AI lives in WhatsApp/Telegram |
| Forgets you each session | Remembers you (files = memory) |
| Locked to one model | Swap Claude / Gemini / GPT freely |
| Your data goes to a vendor's server | Everything runs on your machine |
| Tools are hardcoded | Drop a folder → new skill |

---

## How It Works (in one picture)

```
WhatsApp ──┐                     ┌── Claude
Telegram ──┤── Gateway (Node) ───┤── Gemini
Discord  ──┤    on your laptop   ├── GPT
iMessage ──┘                     └── Pi
```

**One process** connects all your chat apps to all your AI models. Add a new chat app = write one small adapter, not rewrite the agent.

### How a message flows

```
You text on WhatsApp → Gateway → AI agent → tools → reply → back to WhatsApp
```

### Memory = files

```
~/.openclaw/workspace/
  IDENTITY.md   ← who the AI is
  SOUL.md       ← personality
  USER.md       ← who you are
  MEMORY.md     ← long-term notes
```

Plain markdown files. You can read them, edit them, version them, sync them. No database.

### Skills = folders

```
skills/weather/SKILL.md   ← AI reads instructions, calls scripts
skills/github/SKILL.md
```

Adding a skill = `git clone` a folder. Sharing = `cp -r`. Community shares them on **ClawHub**.

---

## The Design Thinking (5 "Why" Questions)

Peter & Mario kept asking the same kind of question:

1. **Why is my AI in a browser?** → Because vendors want lock-in. *Bring AI to where the user already is.*
2. **Why does it forget me?** → Because vendors hoard memory. *Give memory back as plain files.*
3. **Why am I locked to one model?** → Because UI + model are bundled. *Strip the UI, keep the gateway, swap the model.*
4. **Why does data leave my machine?** → Because someone else's server is convenient. *Run locally — laptops are fast enough.*
5. **Why are tools fixed?** → Because plugins are usually compiled in. *Make tools markdown folders anyone can write.*

**The pattern:** find a vendor default, invert it, give the user back ownership.

---

## How To Build Something Like This

If you wanted to build OpenClaw (or anything similar) — the order matters:

1. **Smallest end-to-end loop first.** WhatsApp → laptop → AI → reply. No memory, no tools, no skills yet.
2. **Add files-as-memory.** A markdown file the agent reads each turn. That's it.
3. **Add a persona layer.** `IDENTITY.md`, `SOUL.md`, `USER.md` — agent reads them at session start.
4. **Move tools into folders (Skills).** Agent learns by reading SKILL.md, like a human reading a README.
5. **Make the model swappable.** Same loop, different LLM endpoint.
6. **Add more transports.** Telegram, Discord, iMessage — each is a small adapter, not a rewrite.
7. **Open the ecosystem.** A registry (ClawHub) lets the community grow it without growing your codebase.

**The principle:** start narrow, decouple along the axes where vendors usually lock you in (transport, model, storage, tools), and use formats users can read (markdown > JSON, files > DB rows).

---

## The Philosophy

1. **Own your data** — runs on your hardware
2. **Any platform** — Mac, Linux, Windows, Raspberry Pi
3. **Agent-native** — built for AI agents (tools, memory, sessions), not just chat
4. **Fun over corporate** — space lobster > sterile branding 🦞

---

## Links

- [GitHub](https://github.com/openclaw/openclaw) · [Docs](https://docs.openclaw.ai) · [ClawHub](https://clawhub.com) · [Discord](https://discord.com/invite/clawd)

---

*Last updated: 2026-05-07*
