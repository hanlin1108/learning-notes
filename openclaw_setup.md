# OpenClaw Setup

## Prerequisites

- MacBook with Apple Silicon (M1/M2/M3/M4/M5)
- macOS fully updated
- Anthropic API key (with billing enabled)

-----

## Step 1 — Install Command Line Tools

```bash
xcode-select --install
```

## Step 2 — Install OpenClaw

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

This installs Node.js (if needed) and the OpenClaw CLI.

## Step 3 — Run Onboarding

```bash
openclaw onboard --install-daemon
```

During the wizard, select:

|Prompt                                      |Recommended Choice                          |
|--------------------------------------------|--------------------------------------------|
|Default model                               |`anthropic/claude-sonnet-4-6`               |
|Search provider                             |Your preference (Gemini, Brave, etc.)       |
|Skills dependencies                         |`camsnap`, `peekaboo`, `clawhub`, `mcporter`|
|API keys (Google Places, Notion, ElevenLabs)|Skip (No) — add later if needed             |
|Hooks                                       |`session-memory` + `command-logger`         |
|Gateway runtime                             |Node (default)                              |
|Gateway service                             |Restart or Install                          |
|How to hatch                                |Hatch in TUI                                |

## Step 4 — Get Your Anthropic API Key

1. Go to **console.anthropic.com**
1. Sign up or log in
1. Go to **Settings → Billing** → add credit card and load credits ($5 minimum)
1. Go to **Settings → API Keys** → click **Create Key**
1. Copy the key immediately (starts with `sk-ant-`, shown only once)

## Step 5 — Verify the Key Works

```bash
curl https://api.anthropic.com/v1/messages \
  -H "x-api-key: sk-ant-YOUR-KEY-HERE" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{"model":"claude-sonnet-4-6","max_tokens":50,"messages":[{"role":"user","content":"hi"}]}'
```

You should get a JSON response (not an error).

## Step 6 — Configure the Key in OpenClaw

```bash
openclaw configure --section model
```

Choose Anthropic → select **API key** (not OAuth/account) → paste your key.

## Step 7 — Restart & Test

```bash
openclaw gateway restart
openclaw tui
```

Type a message. If the agent responds, you're good!

## Step 8 — Connect WhatsApp

```bash
openclaw channels login --channel whatsapp
```

Scan the QR code with your phone (WhatsApp → Settings → Linked Devices → Link a Device). If the first attempt fails, run the command again.

## Step 9 — Connect Gmail

```bash
gog auth add your-email@gmail.com --services gmail
```

This opens your browser for Google login. Sign in and authorize access. After that, restart the gateway:

```bash
openclaw gateway restart
```

Now your agent can send and read emails. Test it in the TUI or WhatsApp by saying "Send a test email to your-email@gmail.com".

-----

## Useful Commands

|Command                     |What it does                   |
|----------------------------|-------------------------------|
|`openclaw tui`              |Open the terminal chat UI      |
|`openclaw dashboard`        |Open the web UI in browser     |
|`openclaw gateway status`   |Check if the gateway is running|
|`openclaw gateway start`    |Start the gateway manually     |
|`openclaw gateway stop`     |Stop the gateway               |
|`openclaw gateway restart`  |Restart the gateway            |
|`openclaw gateway install`  |Re-enable auto-start on login  |
|`openclaw gateway uninstall`|Remove auto-start              |
|`openclaw configure`        |Re-run configuration           |
|`openclaw doctor --fix`     |Auto-diagnose and fix issues   |
|`openclaw models status`    |Check model & auth status      |

-----

## Essential Commands

```bash
# Start OpenClaw
openclaw gateway start

# Stop OpenClaw
openclaw gateway stop

# Restart OpenClaw
openclaw gateway restart

# Keep Mac on forever (lid closed, persists after reboot)
sudo pmset -a disablesleep 1

# Stop keeping Mac on forever
sudo pmset -a disablesleep 0

# Keep Mac awake with lid open (resets after reboot)
caffeinate -s &

# Stop caffeinate
killall caffeinate
```

-----

## Good to Know

- **Auto-start:** The gateway runs automatically on login — no manual start needed.
- **Either way:** Keep the MacBook plugged in and in a ventilated spot to avoid heat buildup.
- **API billing:** You pay per token used at console.anthropic.com. Monitor usage under **Settings → Usage**.
- **Switch models on the fly:** Use `/model opus` or `/model sonnet` in the TUI.
- **Always use API key, not OAuth:** OAuth tokens for Anthropic are unreliable with OpenClaw. Stick with a direct API key.
