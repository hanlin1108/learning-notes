# Claude Code Learning Notes

> Claude Code is Anthropic's agentic AI coding tool. It lives in your terminal, reads and writes your actual files, runs shell commands, and works with your existing tools — no new IDE required.

---

## Table of Contents

1. [What Is Claude Code?](#1-what-is-claude-code)
2. [Installation & Setup](#2-installation--setup)
3. [Core Interaction Model](#3-core-interaction-model)
4. [Slash Commands Reference](#4-slash-commands-reference)
5. [File & Codebase Operations](#5-file--codebase-operations)
6. [Memory System: CLAUDE.md](#6-memory-system-claudemd)
7. [Permissions & Safety Model](#7-permissions--safety-model)
8. [MCP Servers](#8-mcp-servers)
9. [GitHub Integration](#9-github-integration)
10. [IDE Integration](#10-ide-integration)
11. [Prompt Patterns That Work](#11-prompt-patterns-that-work)
12. [Common Workflows](#12-common-workflows)
13. [Tips & Gotchas](#13-tips--gotchas)

---

## 1. What Is Claude Code?

Claude Code is a **CLI-based agentic coding assistant** — not a plugin, not a chat window bolted onto an editor. It runs in your terminal and has real access to your machine:

| Capability | What it means |
|---|---|
| Read files | Understands your entire codebase, not just the snippet you paste |
| Write / edit files | Makes real changes directly (with your approval) |
| Run shell commands | Can execute tests, builds, git commands, linters |
| Search code | Searches by content, file name, pattern |
| Call external tools | Connects to databases, APIs, and services via MCP |

**The key mental model**: Claude Code is less like a chatbot and more like a junior developer sitting at your keyboard. You describe the task; it does the work; you review and approve each action.

### When to Use It (vs. Claude.ai Chat)

| Situation | Use |
|---|---|
| Editing real files in a project | Claude Code |
| Running tests, git commands, builds | Claude Code |
| Asking a conceptual question | Either |
| Exploring an idea without touching files | Claude.ai chat |

---

## 2. Installation & Setup

### Prerequisites

- **Node.js** ≥ 18 (check: `node --version`)
- A Claude **Pro or Max subscription**, or an Anthropic API key

### Install

```bash
npm install -g @anthropic-ai/claude-code
```

### Authenticate

Two options:

**Option A: Claude subscription (recommended — no per-token cost)**
```bash
claude          # First launch opens a browser login
                # Sign in with your Claude account
```

**Option B: API key**
```bash
export ANTHROPIC_API_KEY="sk-ant-..."
claude
```

> ⚠️ If `ANTHROPIC_API_KEY` is set in your shell, it takes priority over your subscription and you'll be billed per token. Check with `echo $ANTHROPIC_API_KEY`.

### Launch

```bash
cd your-project
claude          # Start interactive session in this directory
```

Claude Code indexes your current directory on startup.

### Update

```bash
claude update   # Always run this before a big task
```

---

## 3. Core Interaction Model

### Agentic Loops

Unlike a normal chatbot, Claude Code can take **multiple actions in sequence** to complete a task:

```
You: "Fix all TypeScript errors and run the tests"

Claude Code:
  1. Searches for *.ts files with errors
  2. Reads each file
  3. Edits them one by one
  4. Runs: npx tsc --noEmit
  5. Runs: npm test
  6. Reports what it did and what passed/failed
```

Each action that touches your system requires approval — you see exactly what it will do before it happens.

### Approval Modes

| Mode | Behavior |
|---|---|
| **Default** | Ask before every file write or shell command |
| `--dangerously-skip-permissions` | Skip all prompts (only for trusted automated environments) |
| Allowlist in settings | Pre-approve specific safe commands (e.g. `npm test`) |

### Context Window

Claude Code reads files into its context window as needed. For large codebases:
- Use `/compact` to summarize long conversations and free up space
- Be explicit: "only look at `src/auth/`" limits what it reads

---

## 4. Slash Commands Reference

Type `/` in the Claude Code prompt to see all available commands.

### Most Useful

| Command | What it does |
|---|---|
| `/help` | List all commands |
| `/clear` | Start a fresh conversation (clears context) |
| `/compact` | Summarize conversation history to save context space |
| `/init` | Generate a `CLAUDE.md` file for your project |
| `/review` | Run a code review on your current changes |
| `/cost` | Show token usage and estimated cost for this session |
| `/quit` or `Ctrl+C` | Exit Claude Code |

### GitHub-Specific

| Command | What it does |
|---|---|
| `/install-github-app` | Set up the GitHub App for cloud-based @claude usage |
| `/pr-comments` | Pull in comments from a GitHub PR for context |

### Config

| Command | What it does |
|---|---|
| `/allowed-tools` | View which tools are currently enabled |
| `/model` | Switch the Claude model for this session |

---

## 5. File & Codebase Operations

Claude Code has built-in tools for working with files. You don't call these directly — Claude uses them automatically when you make requests.

### What Claude Can Do

```
Read a file:          Read the contents of any file
Edit a file:          Make precise, targeted edits (not full rewrites)
Write a new file:     Create files from scratch
Search by name:       Find files matching a glob pattern (e.g. **/*.test.ts)
Search by content:    Find all files containing a string or regex
Run shell commands:   Execute any terminal command
```

### Useful Prompts for File Work

```
"Read package.json and explain the project structure"
"Find all files that import from ./utils/auth"
"Rename the function processData to transformPayload everywhere"
"Create a new file src/utils/date.ts with a formatDate helper"
"Show me all TODO comments in the codebase"
```

### Large Codebases

For big repos, help Claude stay focused:
- "Only look at files in `src/api/`"
- "Here's the relevant file: [paste path]"
- Use `/compact` regularly to keep context efficient

---

## 6. Memory System: CLAUDE.md

`CLAUDE.md` is a special file Claude Code reads at startup. Think of it as **persistent instructions for your project** — it tells Claude how to work with your specific codebase.

### What to Put in It

```markdown
# CLAUDE.md

## Project Overview
This is a Next.js 14 app with a Postgres database and Prisma ORM.

## Commands
- `npm run dev` — start dev server (port 3000)
- `npm test` — run Jest tests
- `npm run lint` — ESLint
- `npx prisma studio` — open DB browser

## Coding Conventions
- Use named exports (not default exports)
- All API routes live in `src/app/api/`
- Database queries go in `src/lib/db/`
- Error handling: always use the AppError class from `src/lib/errors.ts`

## Things to Avoid
- Don't modify `src/generated/` — auto-generated by Prisma
- Don't add console.log statements to production code
```

### Where to Put It

| Location | Scope |
|---|---|
| `/project-root/CLAUDE.md` | Applies to this project, committed to the repo |
| `~/.claude/CLAUDE.md` | Applies to all projects globally |
| `src/CLAUDE.md` | Applies only when working in `src/` |

> **Best practice**: Commit CLAUDE.md to the repo. Every teammate (and Claude in GitHub Actions) will benefit from the same context.

### Auto-Memory

As you work, Claude Code can also save notes about your preferences and project patterns. These are stored in `~/.claude/` and recalled in future sessions.

---

## 7. Permissions & Safety Model

Claude Code is designed to be powerful *and* safe. It will always show you what it's about to do before doing it.

### The Permission Flow

```
Claude wants to run: git push origin main

? Allow this command? (y/n/always/never)
  y      → allow once
  n      → deny
  always → add to allowlist (never ask again)
  never  → add to blocklist
```

### Settings File

Persistent permissions live in `.claude/settings.json` (project-level) or `~/.claude/settings.json` (global).

```json
{
  "permissions": {
    "allow": [
      "Bash(npm test)",
      "Bash(npm run lint)",
      "Bash(git status)",
      "Bash(git diff)"
    ],
    "deny": [
      "Bash(git push --force)"
    ]
  }
}
```

### What Claude Won't Do By Default

- Force-push to main
- Delete files without confirmation
- Run commands outside the current project directory
- Modify `.github/workflows/` (when running as a GitHub App)

---

## 8. MCP Servers

**MCP (Model Context Protocol)** is the standard for connecting Claude Code to external tools. An MCP server is a small process that gives Claude Code new capabilities.

### What MCP Unlocks

| MCP Server | What Claude Can Do |
|---|---|
| GitHub MCP | Read issues, PRs, comments; post comments |
| PostgreSQL MCP | Query your database directly |
| Brave Search MCP | Search the web in real time |
| Filesystem MCP | Access files outside the current project |
| Slack MCP | Read and post to Slack |
| Custom MCP | Anything you build |

### Add an MCP Server

```bash
claude mcp add <server-name>         # Interactive setup
claude mcp add github                # e.g. GitHub MCP
claude mcp list                      # See installed servers
claude mcp remove <server-name>      # Remove a server
```

MCP config is stored in `~/.claude/claude.json` or `.mcp.json` at the project root.

### Build Your Own MCP Server

MCP servers can be written in any language. A Python MCP server that exposes a `get_weather` tool is under 50 lines. See the [MCP specification](https://modelcontextprotocol.io) for details.

---

## 9. GitHub Integration

See [Connecting Claude Code to GitHub](./connect_claude_code_to_github.md) for the full setup guide. Quick summary:

### Method A: Local Terminal
Run `claude` locally, give it git tasks, it handles add/commit/push with your approval.

### Method B: GitHub App (@claude in issues/PRs)
- Install once with `/install-github-app`
- Mention `@claude` in any issue or PR comment
- Claude spins up in GitHub Actions, does the work, and opens a branch/PR
- Your computer doesn't need to be on

### What Claude Can Do via GitHub App

```
@claude Fix the typo in README.md
@claude Add unit tests for the auth module
@claude Implement the feature described in this issue
@claude Review this PR for security issues
@claude Refactor the UserService to use async/await
```

### GitHub App Workflow

```
Issue with @claude mention
        ↓
GitHub Actions triggers Claude Code
        ↓
Claude reads repo → creates branch → makes changes → pushes
        ↓
You click "Create PR" link in Claude's comment
        ↓
Review the diff → merge or iterate with more @claude comments
```

---

## 10. IDE Integration

Claude Code integrates directly into popular IDEs while still running from the terminal.

### VS Code

Install the **Claude Code extension** from the VS Code marketplace.

Benefits:
- See Claude's file edits highlighted inline before approving
- Run Claude Code from within VS Code's terminal
- Diff view shows exactly what changed

### JetBrains (IntelliJ, PyCharm, etc.)

Same concept — install the **Claude Code plugin** from the JetBrains marketplace.

### The Key Insight

Claude Code is **editor-agnostic**. The CLI is the core; IDE extensions are a convenience layer. Everything works without them — they just improve the review/approval experience.

---

## 11. Prompt Patterns That Work

### Be Specific About Scope

```
❌ "Fix the authentication"
✅ "Fix the JWT token expiry bug in src/auth/verify.ts — tokens are not refreshing correctly when the user has the 'remember me' option enabled"
```

### Give It a Goal, Not Steps

```
❌ "Open verify.ts, find the refreshToken function, add a check for..."
✅ "The refresh token flow is broken. Make it work correctly, run the tests to confirm."
```

Claude Code is better than you at figuring out the steps. Give it the outcome.

### Reference Files Explicitly When Needed

```
"Look at src/api/users.ts and src/models/User.ts — the API response shape doesn't match the model"
```

### Ask for a Plan First (For Big Changes)

```
"Before making any changes: what's your plan for migrating the database from MySQL to PostgreSQL? List the files you'll touch."
```

### Iterating on Output

```
"That's good but the error messages are too technical for end users. Rewrite them to be friendly."
"The test you wrote doesn't cover the edge case where userId is null."
```

---

## 12. Common Workflows

### Debug a Failing Test

```
"The test UserService.createUser_withDuplicateEmail is failing. Figure out why and fix it."
```

Claude will read the test, read the implementation, trace the failure, and fix it.

### Add a New Feature

```
"Add a /api/users/:id/avatar endpoint that accepts a multipart form upload, saves to S3, and updates the user record. Follow the patterns in the existing /api/users endpoints."
```

### Code Review Before Committing

```
"Review my staged changes. Look for bugs, security issues, and anything that doesn't follow our coding conventions in CLAUDE.md."
```

### Refactor

```
"Refactor the payment module (src/payments/) to remove code duplication. Don't change any behavior — just clean it up. Run tests before and after to confirm nothing breaks."
```

### Understand Unfamiliar Code

```
"Walk me through how a user login request flows through this codebase, from the API endpoint to the database query."
```

### Write Tests

```
"Write unit tests for the PricingCalculator class. Cover the happy path and the edge cases: zero quantity, negative prices, and currency conversion."
```

---

## 13. Tips & Gotchas

### Do

- ✅ **Run `claude update` regularly** — new features ship frequently
- ✅ **Invest in a good CLAUDE.md** — 20 minutes writing it saves hours of re-explaining conventions
- ✅ **Use `/compact` on long sessions** — keeps responses fast and on-topic
- ✅ **Let Claude run tests** — "fix it and make the tests pass" produces better results than "fix this specific line"
- ✅ **Review diffs carefully** — Claude is fast but not infallible; you're the final check

### Avoid

- ❌ **Pasting huge chunks of code in the prompt** — just give the file path, Claude will read it
- ❌ **Vague requests on large codebases** — scope your request to a module or file
- ❌ **Ignoring the permission prompts** — they exist for a reason; read what Claude is about to run
- ❌ **Setting `--dangerously-skip-permissions` on production systems** — reserved for CI/CD with strict controls

### Context Window Management

Claude Code's context is finite. Signs you're running out:
- Responses get slower or vaguer
- Claude starts "forgetting" earlier parts of the task

Fixes:
- `/compact` — compresses history while keeping the key facts
- `/clear` — fresh start (loses conversation history)
- Start a new session with a focused scope

### Cost Awareness

- Max/Pro subscription: Claude Code usage draws from your plan's usage quota
- API key: billed per token — check `/cost` regularly during long sessions
- Expensive operations: large file reads, repeated tool calls in loops

---

## The Core Philosophy

> Claude Code is not autocomplete. It's an agent — give it a goal and let it figure out the path.

The shift in thinking:

| Old way | Claude Code way |
|---|---|
| "Which function do I call?" | "Make the payment flow work end to end" |
| "How do I fix this syntax error?" | "Fix the failing tests in the payments module" |
| "What's the regex for email validation?" | "Add proper email validation to the signup form" |

The more you describe the *outcome* you want and the *constraints* that matter, the better Claude Code performs.

---

*Last updated: 2026-04-21*
