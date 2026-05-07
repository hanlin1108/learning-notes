# Connect OpenClaw to Claude Code

## Architecture

```
OpenClaw (daily chat, automation) ←→ Claude Code (coding, deep tasks)
                       ↕
             MCP Gmail (read + send)
```

Switch between them with `/model` in chat. No restart needed.

---

## 1. Claude Code as CLI Backend

### Prerequisites

```bash
npm install -g @anthropic-ai/claude-code
claude auth login
claude auth status --text
# Should show: Authenticated
```

### Edit Config

```bash
code ~/.openclaw/openclaw.json
```

Replace the entire `agents` section with:

```json
"agents": {
  "defaults": {
    "model": {
      "primary": "anthropic/claude-sonnet-4-6",
      "fallbacks": [
        "anthropic/claude-opus-4-6"
      ]
    },
    "models": {
      "anthropic/claude-sonnet-4-6": { "alias": "openclaw" },
      "anthropic/claude-opus-4-6": { "alias": "openclawopus" },
      "claude-cli/claude-sonnet-4-6": { "alias": "claude" },
      "claude-cli/claude-opus-4-6": { "alias": "claudeopus" }
    },
    "workspace": "/Users/lihanlin/.openclaw/workspace",
    "compaction": { "mode": "safeguard" },
    "maxConcurrent": 4,
    "subagents": { "maxConcurrent": 8 }
  }
}
```

**Do NOT add `agents.list` — it has known bugs with multi-agent routing. Use `/model` switching only.**

### Restart & Test

```bash
openclaw gateway restart
```

Then in OpenClaw chat:

```
# Verify: openclaw, openclawopus, claude, claudeopus
/model claude              # Switch to Claude Code (Sonnet)
Hello, which model are you?  # Test it
/model openclaw            # Switch back
```

### Switch Commands

| Command               | Routes to                                    |
|-----------------------|----------------------------------------------|
| `/model openclaw`     | OpenClaw API — Sonnet (default, fast, cheap) |
| `/model openclawopus` | OpenClaw API — Opus                          |
| `/model claude`       | Local Claude Code — Sonnet                   |
| `/model claudeopus`   | Local Claude Code — Opus                     |
| `/model status`       | Check current model                          |
| `/new`                | New session, resets to default (openclaw)    |

All switches are sticky — stays on that model until you switch again or start a new session.

---

## 2. Gmail MCP (IMAP/SMTP)

### Generate App Password

1. Go to <https://myaccount.google.com/apppasswords>
2. Enable 2-Step Verification first if not already on
3. Create app password for "Claude Code"
4. Copy the 16-char password (shown once only)

### Add to Claude Code

```bash
claude mcp add imap-email -s user \
  -e IMAP_USER=your@gmail.com \
  -e IMAP_PASSWORD='your-16-char-app-password' \
  -e IMAP_HOST=imap.gmail.com \
  -e SMTP_HOST=smtp.gmail.com \
  -- npx -y imap-email-mcp
```

### Verify

```bash
claude          # Restart Claude Code
/mcp            # Should show: imap-email ✔ connected, 10 tools
```

Test: ask Claude Code to send a test email to yourself. Gmail MCP also works when OpenClaw routes through Claude Code (`/model claude`).

---

## 3. Key Notes

- **OpenClaw vs Claude Code**: Claude Code is stronger for coding (lower hallucination, self-correction loops). OpenClaw is better for daily automation (scheduled tasks, messaging platforms, email triage)
- **Output filtering**: When using Claude Code through OpenClaw, you only get the final text — no intermediate steps. Check `~/.claude/projects/` for full session logs
- **Cost**: OpenClaw API = pay per token. Claude Code CLI = uses your Pro/Max subscription
- **Config files**:
  - OpenClaw: `~/.openclaw/openclaw.json`
  - Claude Code MCP: `~/.claude.json`
- **Google's official Gmail MCP cannot send email** — that's why we use `imap-email-mcp` (third-party, IMAP/SMTP)
