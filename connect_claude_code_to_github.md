# Connecting Claude Code to GitHub

> Goal: Use Claude Code locally to make changes and push them directly to GitHub.

---

## How It Works

Claude Code has no built-in GitHub magic. It does two things:

1. **Edits code on your machine** (reads and writes files)
2. **Runs git commands for you** (`git add`, `git commit`, `git push`)

So "connecting to GitHub" means: **git can push to GitHub** + **Claude Code has permission to run those commands**.

**Think of Claude Code as your git assistant.**

---

## One-Time Setup (3 Steps)

### 1. Configure git identity
```bash
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
```

### 2. Authenticate with GitHub (recommended: `gh` CLI)
```bash
brew install gh          # Mac
gh auth login            # Follow prompts: GitHub.com → HTTPS → browser login
```
After login, `git push` and PR creation work without a password.

### 3. Link your local repo to GitHub

**A. Existing GitHub repo** → clone it
```bash
git clone https://github.com/your-username/repo-name.git
cd repo-name
```

**B. Local project** → push to a new repo
```bash
cd your-project
git init
gh repo create           # Creates and links a GitHub repo in one command
```

---

## Method A: Local Terminal (Simplest — Recommended for Beginners)

1. Log in once: `gh auth login`
2. Open Claude Code in your project folder: `claude`
3. Include GitHub actions directly in your request

**Example prompts:**
- "Add a login page on the `feature/login` branch, commit and push to origin"
- "Summarize the last 5 commits on the main branch"
- "Fix the bug in issue #23, run the tests, then open a PR"

Claude Code will run `git add`, `git commit`, `git push`, and `gh pr create` automatically. It asks for your confirmation before touching the repo — just review and approve.

---

## Method B: GitHub App (Let Claude Work Directly on GitHub)

### Core Concept

Mention `@claude` in any GitHub issue or PR, and Claude will automatically spin up in the GitHub cloud to read code, write code, and open PRs — your computer can stay off.

### One-Time Setup (5 minutes)

1. Run `claude update` in terminal, confirm version ≥ 1.0.44
2. Navigate to your project: `cd your/project/path`
3. Launch Claude Code: `claude`
4. Run the install command: `/install-github-app`

> **Important:** If `ANTHROPIC_API_KEY` is set in your environment, Claude Code uses the API key instead of your Max subscription (this incurs API charges).
> Check: `echo $ANTHROPIC_API_KEY`. If it returns a value, remove it from `~/.zshrc` / `~/.bashrc` and restart your terminal.

The setup wizard will walk you through:

| Step | What to do |
|------|-----------|
| Authorize GitHub App in browser | Select target repo → Install |
| Choose workflows | Check both (@claude + auto review) |
| Choose authentication | Select "Create a long-lived token with your Claude subscription" (uses your Max plan — no extra cost) |
| Final step | A setup PR is auto-generated |

Go to GitHub and **Merge that setup PR** → activation complete ✅

### What You Get After Installation

A new `.github/workflows/` folder appears in your repo containing:
- `claude.yml` — handles @claude mentions
- `claude-code-review.yml` — auto-reviews new PRs (delete if not needed)

Three OAuth tokens are added to your repo Secrets (managed automatically — don't touch).

### Daily Usage (3 Steps)

1. Go to repo → Issues → New issue
2. Write `@claude` + your task description
3. Wait 1–3 minutes — Claude will open a PR

**Example prompts:**
```
@claude Fix the typos in README
@claude Add English docstrings to all functions in utils.py
@claude Implement the login feature described in issue #12
```

### Full Workflow

```
You write an issue with @claude
        ↓
GitHub spins up a temporary VM
        ↓
Claude reads code → creates a branch → makes changes → pushes
        ↓
You click "Create PR" to open the pull request
        ↓
Review the diff → Merge if happy,
                  or comment @claude to iterate further
        ↓
main branch updated ✅
```

### Key Points

- Changes always live on a new branch — main is never touched directly. You can always back out before merging.
- Claude doesn't auto-open PRs — after it finishes, you click "Create PR" yourself.
- PRs can be iterated infinitely — not satisfied? Just `@claude` again, the same PR updates automatically.
- Shared Max quota — Claude's work on GitHub, your local Claude Code, and Claude on the web all draw from the same Max subscription pool.

> **Enabling @claude in Other Repos Later:** Run `/install-github-app` in each repo (recommended)

---

## Method A vs Method B

| | Method A (Local Terminal) | Method B (GitHub App) |
|---|---|---|
| Where Claude runs | Your computer | GitHub cloud |
| Do you need to be present? | Yes | No — even mobile issues work |
| Response speed | Seconds | 1–3 minutes |
| Best for | Exploring, debugging, learning | Defined tasks, docs, background automation |
| Rule of thumb | Need to think → Method A | Need to execute → Method B |

---

## Core Workflow

```
1. Describe what you want: "Add a section about X to xxx.md"
2. After Claude Code edits the file, say: "Commit and push to GitHub"
3. It runs git add / commit / push — approve on first use
```

You never need to type git commands manually.

---

## Prompt Templates

| Goal | Tell Claude Code |
|------|-----------------|
| Save and push | "commit and push to GitHub" |
| New branch + PR | "create a branch `xxx`, make the changes, push and open a PR" |
| Sync from remote | "git pull first, then start" |
| Check status | "What's the current repo status? Any uncommitted changes?" |
| Undo last commit | "Undo the last commit (not pushed yet)" |

---

## Common Pitfalls

### 1. `.env` / API keys accidentally pushed
Always create a `.gitignore` first and tell Claude Code: "never commit sensitive files."

### 2. Working on the wrong branch
Ask before starting: "What branch am I on?"

### 3. Push fails due to permissions
On the first push, Claude Code will request permission to run git commands — **allow it**. To avoid repeated prompts, add common git commands to the allowlist in `.claude/settings.json`.

---

## 30-Second Checklist

- [ ] `git config --global user.email` is set?
- [ ] `gh auth status` shows logged in?
- [ ] `git remote -v` points to the correct GitHub repo?
- [ ] `.gitignore` blocks `.env` and large files?

All checked → let Claude Code get to work.
