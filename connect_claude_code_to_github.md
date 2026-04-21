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

### Step 1. Update Claude Code
```bash
claude update
claude --version
```
Version must be ≥ 1.0.44.

### Step 2. Log in with your Max account
```bash
claude
```
Type `/status` to check the current account. If it's not Max, run `/logout` then `/login` with your Max account.

> **Important:** If `ANTHROPIC_API_KEY` is set in your environment, Claude Code uses the API key instead of your Max subscription (this incurs API charges).
> Check: `echo $ANTHROPIC_API_KEY`. If it returns a value, remove it from `~/.zshrc` / `~/.bashrc` and restart your terminal.

### Step 3. Navigate to your project
```bash
cd your-project-path
claude
```

### Step 4. Install the GitHub App
```
/install-github-app
```

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
