# OpenClaw Learning Notes

> OpenClaw is a free, open-source, self-hosted personal AI assistant that runs on your local machine and lets you chat with AI through messaging apps like Discord, Telegram, and Slack.

---

## Table of Contents

1. [What Is OpenClaw?](#1-what-is-openclaw)
2. [Why It's Popular](#2-why-its-popular)
3. [What Can It Do?](#3-what-can-it-do)
4. [How It Works (The Architecture)](#4-how-it-works-the-architecture)
5. [How to Get Started](#5-how-to-get-started)
6. [Common Use Cases](#6-common-use-cases)
7. [Tips & Key Concepts](#7-tips--key-concepts)
8. [The 20% → 80% Summary](#8-the-20--80-summary)

---

## 1. What Is OpenClaw?

OpenClaw is a **self-hosted AI assistant** — meaning you run it yourself, on your own hardware, with your own API keys.

Instead of using a web app to talk to AI, OpenClaw lets you chat through apps you already use every day: **Discord, Telegram, and Slack**.

| | Cloud AI (ChatGPT, Claude.ai) | OpenClaw |
|---|---|---|
| Where it runs | Company's servers | **Your machine** |
| Who owns the data | The company | **You** |
| Customizable | Limited | **Fully customizable** |
| Cost | Subscription | **Free (you pay only for API usage)** |
| Interface | Web app | **Discord / Telegram / Slack bots** |

**In one sentence**: OpenClaw turns your local machine into a personal AI server that you control completely, accessible through your favorite chat apps.

---

## 2. Why It's Popular

### Privacy First
Your conversations never leave your machine. No third party stores your chat history or uses it for training.

### Freedom of Choice
You can plug in **any AI model** — cloud APIs (OpenAI, Anthropic, Google) or local models (Ollama, LM Studio). Switch models without changing your workflow.

### Already in Your Chat Apps
You don't need to open a new app. Ask your AI in the same Discord server you already use, or ping it on Telegram.

### Free and Open Source
No subscription. No vendor lock-in. Anyone can inspect the code, contribute, or fork it.

### Designed for 2026 AI Workflows
Built with modern AI patterns in mind — multi-model support, tool use, memory, and integrations are first-class features, not afterthoughts.

---

## 3. What Can It Do?

### Core Capabilities

| Feature | What It Means |
|---|---|
| **Multi-platform bots** | Talk to your AI via Discord, Telegram, or Slack |
| **Any LLM backend** | Connect to OpenAI, Anthropic, local models via Ollama, etc. |
| **Conversation memory** | Remembers context across sessions |
| **Tool / function use** | AI can search the web, run code, read files |
| **Persistent configuration** | Set personality, system prompts, and behaviors per server/channel |
| **Multi-user support** | Multiple people on the same Discord server can use it |
| **Self-hosted = private** | Your data stays on your machine |

### What It's NOT
- Not a cloud service — it requires you to run a server (your own computer or a VPS)
- Not a replacement for the AI model itself — you still need API keys or local models

---

## 4. How It Works (The Architecture)

Think of OpenClaw as a **bridge** between your chat app and an AI model.

```
You (Discord/Telegram/Slack)
        ↓
   OpenClaw Bot
        ↓
   Your AI Backend
  (OpenAI / Claude / Ollama)
        ↓
   Response back to chat
```

### Key Pieces

**1. The Bot Layer**
OpenClaw registers bots on your platforms. When you send a message, the bot catches it.

**2. The Orchestrator**
OpenClaw processes your message — applies your system prompt, retrieves memory if needed, decides what tools to call.

**3. The LLM Connector**
Sends the assembled request to whatever AI backend you've configured, gets the response.

**4. Memory Store**
Persists conversation history locally so the AI feels consistent across sessions.

**5. Tool Integrations**
Optional tools (web search, code execution, file reading) that the AI can invoke when needed.

---

## 5. How to Get Started

### Prerequisites
- A machine running 24/7 (your PC, a Raspberry Pi, or a cheap VPS)
- API keys for your chosen AI (e.g., OpenAI, Anthropic) OR a local model via Ollama
- A Discord/Telegram/Slack account to create a bot

### High-Level Setup Steps

```
1. Clone the OpenClaw repo from GitHub
2. Configure your .env file:
   - Add your AI API key (or point to local Ollama endpoint)
   - Add your Discord/Telegram/Slack bot token
3. Set your system prompt and preferences in config
4. Run OpenClaw (usually: python main.py or docker compose up)
5. Invite the bot to your Discord server / start your Telegram bot
6. Start chatting
```

### The Config File (What Matters Most)

The config is where most of the power lives:

| Config Key | What It Controls |
|---|---|
| `model` | Which AI model to use (gpt-4o, claude-3-5-sonnet, etc.) |
| `system_prompt` | The AI's personality and core instructions |
| `memory_enabled` | Whether it remembers past conversations |
| `tools` | Which tools (web search, code, etc.) to enable |
| `platforms` | Which chat platforms are active |

---

## 6. Common Use Cases

### Personal AI Assistant
Your own AI available 24/7 in Discord — ask it anything, get help with tasks, have it draft emails or summarize articles.

### Team AI Bot
Add it to a shared Discord server so your team can all query the same AI assistant with shared context.

### Developer Productivity
Use it with coding-capable models (Claude, GPT-4o) to review code, explain errors, or generate snippets — all from your Slack channel.

### Private Research Assistant
Because all data stays local, it's ideal for sensitive work where you can't use cloud AI tools.

### Learning & Experimentation
Perfect for developers who want to learn how AI systems, bots, and integrations work in practice.

---

## 7. Tips & Key Concepts

### The System Prompt Is Everything
The most impactful thing you can customize is the system prompt. A well-crafted system prompt makes the assistant dramatically more useful for your specific needs.

### Start Simple, Add Tools Later
Begin with just the chat capability. Once that's stable, layer in tools (web search, code execution) one at a time.

### Local Models vs. API Models

| | Local (Ollama) | API (OpenAI/Anthropic) |
|---|---|---|
| Cost | Free (after hardware) | Pay per token |
| Speed | Depends on your GPU | Fast |
| Privacy | 100% local | Data sent to provider |
| Capability | Smaller models | Latest/most capable |

Use **API models** when you need maximum capability. Use **local models** when privacy is critical or you want zero API costs.

### Memory Has a Context Window
Even with memory enabled, there's a limit to how much history the AI can "see" at once. OpenClaw handles this by summarizing older conversations.

### Bot Token ≠ API Key
You need two separate credentials: one for your chat platform (Discord bot token) and one for your AI provider (OpenAI key). Don't confuse them.

---

## 8. The 20% → 80% Summary

If you remember only four things about OpenClaw:

1. **It's a bridge** — sits between your chat apps and an AI model, running on hardware you own.

2. **Self-hosted = privacy + control** — your data never leaves your machine; you choose the model, the personality, the tools.

3. **The config file is your power lever** — model choice, system prompt, memory, and tools are all set there. Master the config and you master OpenClaw.

4. **Start with the basics** — get the bot talking in Discord/Telegram first. Add memory, tools, and multi-model support once the core works.

> **The big idea**: OpenClaw lets you run a fully private, fully customizable AI assistant through apps you already use — no subscriptions, no data sharing, no walled gardens.