# Connect OpenClaw to Claude Code

## Architecture

```
OpenClaw (daily chat, automation)
        ↕ or @claude
    Claude Code (coding, deep tasks)
        ↕
    MCP Gmail (read + send email)
```

---

## 1. Claude Code as CLI Backend

### Prerequisites

```bash
# Ensure Claude Code is installed and logged in
npm install -g @anthropic-ai/claude-code
claude auth login
claude auth status --text
# Should show: Authenticated
```

### Edit Config

```bash
code ~/.openclaw/openclaw.json
```

In the `agents` section, replace or merge to match this:

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
      "anthropic/claude-opus-4-6": { "alias": "opus" },
      "anthropic/claude-sonnet-4-6": { "alias": "sonnet" },
      "claude-cli/claude-opus-4-6": { "alias": "claude" }
    },
    "workspace": "/Users/lihanlin/.openclaw/workspace",
    "compaction": { "mode": "safeguard" },
    "maxConcurrent": 4,
    "subagents": { "maxConcurrent": 8 }
  },
  "list": [
    {
      "id": "claude",
      "model": { "primary": "claude-cli/claude-opus-4-6" }
    }
  ]
}
```

### Restart & Test

```bash
# 1. Restart gateway
openclaw gateway restart

# 2. In OpenClaw chat, verify models loaded
/model list
# → Should see: claude, sonnet, opus

# 3. Switch to Claude Code
/model claude

# 4. Test it
Hello, which model are you?

# 5. Switch back to OpenClaw
/model sonnet
```

### Switch Commands

| Action                         | Command          |
|--------------------------------|------------------|
| Switch to Claude Code          | `/model claude`  |
| Switch back to OpenClaw        | `/model sonnet`  |
| One-shot task                  | `@claude <task>` |
| Check current model            | `/model status`  |
| New session (resets to default)| `/new`           |

---

## 2. Gmail MCP (IMAP/SMTP)

### Generate App Password

<https://myaccount.google.com/apppasswords> → create one for "Claude Code"

### Add to Claude Code

```bash
claude mcp add imap-email -s user \
  -e IMAP_USER=your@gmail.com \
  -e IMAP_PASSWORD='your-16-char-app-password' \
  -e IMAP_HOST=imap.gmail.com \
  -e SMTP_HOST=smtp.gmail.com \
  -- npx -y imap-email-mcp
```

Config stored at: `~/.claude.json`

### Verify

```bash
# Restart Claude Code
claude

# Check MCP status
/mcp
# → imap-email should show ✔ connected, 10 tools
```

---

## 3. Key Notes

- **OpenClaw default** → API (fast, cheap). **Claude Code** → local CLI (stronger for coding)
- `/model claude` is sticky (stays until you switch back). `@claude` is one-shot
- `/new` always resets to default OpenClaw API mode
- Gmail MCP works in both direct Claude Code and OpenClaw→Claude Code mode
- Claude Code output is filtered by OpenClaw — you get final text only, not intermediate steps
- Session logs at `~/.claude/projects/` if you need full details
- Config files:
  - OpenClaw: `~/.openclaw/openclaw.json`
  - Claude Code MCP: `~/.claude.json`
