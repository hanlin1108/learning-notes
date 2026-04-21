# Claude Code + GitHub Integration

## Method A: Use Claude Code in Your Local Terminal (Recommended for Beginners)

**Setup (one time):**
```bash
gh auth login
```

**Usage:**
1. Open your project folder in the terminal
2. Run `claude` to start Claude Code
3. Include GitHub actions directly in your prompts

**Example prompts:**
- "Add a login page on the `feature/login` branch, commit and push to origin"
- "Summarize the last 5 commits on the main branch"
- "Fix the bug in issue #23, run the tests, then open a PR"

Claude Code automatically runs `git add`, `git commit`, `git push`, and `gh pr create` for you. It always asks for your confirmation before modifying the repository.

---

## Method B: GitHub App — Let Claude Act Directly on GitHub

### Step 1: Update Claude Code

```bash
claude update
claude --version
```

Required version: **≥ 1.0.44**

### Step 2: Log in with Your Max Account

```bash
claude
```

Inside Claude Code, type `/status` to check the current account. If it's not your Max account, run `/logout` then `/login`.

> ⚠️ **Important:** If `ANTHROPIC_API_KEY` is set as an environment variable, Claude Code will use the API key instead of your Max subscription, which incurs extra charges.
>
> Check: `echo $ANTHROPIC_API_KEY`
>
> If output appears, remove the variable from `~/.zshrc` or `~/.bashrc`, then restart your terminal.

### Step 3: Navigate to Your Project

```bash
cd /path/to/your/project
claude
```

### Step 4: Install the GitHub App

```
/install-github-app
```

Follow the prompts to connect Claude Code to your GitHub repository.
