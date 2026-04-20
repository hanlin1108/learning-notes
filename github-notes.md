# GitHub Learning Notes (Personal v1)

> Compiled from an in-depth conversation with Claude. My starting point for learning Git/GitHub.

---

## Table of Contents

1. [Core Concepts](#1-core-concepts)
2. [The Four "Places": How Code Flows](#2-the-four-places-how-code-flows)
3. [The Essence of Commit and Push](#3-the-essence-of-commit-and-push)
4. [Branches: Git's Killer Feature](#4-branches-gits-killer-feature)
5. [Two Main Workflows](#5-two-main-workflows)
6. [Command Cheat Sheet (by Frequency)](#6-command-cheat-sheet-by-frequency)
7. [Scenario Playbook](#7-scenario-playbook)
8. [Common Confusions Clarified](#8-common-confusions-clarified)
9. [AI Engineer–Specific Notes](#9-ai-engineerspecific-notes)
10. [Work vs Personal Account Boundaries](#10-work-vs-personal-account-boundaries)
11. [6 Modes for Learning via GitHub](#11-6-modes-for-learning-via-github)
12. [30-Day Kickoff Plan](#12-30-day-kickoff-plan)

---

## 1. Core Concepts

**Git ≠ GitHub**

- **Git** (version control tool): Runs locally on your computer, manages code history.
- **GitHub** (hosting platform): A website that hosts Git repositories, with added collaboration features.

Analogy: Git is Word's "track changes" feature; GitHub is Google Drive.

**7 Concepts You Must Know**

| Concept | Description |
|---|---|
| Repository / Repo | A project folder + its complete history |
| Commit | A code snapshot, with a unique hash and message |
| Branch | An independent line of development; default is `main` |
| Clone | Download a remote repo fully to your local machine |
| Fork | Copy someone else's GitHub repo into your own account (GitHub-only feature) |
| Pull Request (PR) | A request to merge your changes into another branch (GitHub-only) |
| Merge | Combine changes from one branch into another |

**Fork vs Clone**:
- Clone = download to your local machine
- Fork = copy to your own GitHub account (required when you don't have write access)

---

## 2. The Four "Places": How Code Flows

```
Working Directory → Staging Area → Local Repo → Remote Repo (GitHub)
     edit             git add       git commit     git push
                                 ← git pull
```

- **Working Directory**: Files you're currently editing
- **Staging Area**: Files marked with `git add` as "ready to commit"
- **Local Repo**: After `git commit`, changes enter your local history
- **Remote Repo**: After `git push`, changes sync to GitHub

---

## 3. The Essence of Commit and Push

### Commit = A Local Save Point

Think of a video game save system:
- Each commit is a complete "snapshot" of your code
- Has a unique ID (hash), message, timestamp, author
- **Permanently** recorded in your local history

**Commits form a chain**: Each commit knows its parent, forming a linked list.

```
[ morning commit ] ← [ noon commit ] ← [ afternoon commit ]
                                              ↑
                                         latest here
```

### Push = Sync All New Local Commits to GitHub

Key understanding:
- Push is **NOT** just the most recent commit
- Push **IS** "all commits you have locally that the remote doesn't yet have," pushed together **in order**
- After pushing, each commit is **independently preserved** on GitHub

**Example day's workflow**:
```
10:00  git commit -m "Set up skeleton"       ← save point 1
14:00  git commit -m "Add age features"       ← save point 2
16:00  git commit -m "Add location features"  ← save point 3
17:00  git push                               ← push all 3 at once
```

### Why Commit Multiple Times?

1. **Precise rollback**: Undo one step while keeping others
2. **Readable history**: You'll still understand your own work 6 months later
3. **Works offline**: Commit is purely local; only push needs network

### Undoing a Pushed Commit

- Already pushed: Use `git revert <commit-id>` (**adds a new undo commit**, preserves history)
- Not yet pushed: Use `git reset --soft HEAD~1`

**Golden Rule**: **Private history can be rewritten. Public history can only be added to.**

---

## 4. Branches: Git's Killer Feature

### A Branch = A Parallel Universe

- `main` is the primary universe, must stay stable
- Every new change → create a parallel universe (branch)
- Success → merge back via PR
- Failure → discard the branch, main universe untouched

### `git checkout` Old vs New Commands

| Old command | New command (preferred) | Purpose |
|---|---|---|
| `git checkout <branch>` | `git switch <branch>` | Switch branch |
| `git checkout -b <branch>` | `git switch -c <branch>` | Create and switch to new branch |
| `git checkout <file>` | `git restore <file>` | Discard changes in a file |

### When to Create a Branch?

| Scenario | Fork? | Branch? |
|---|---|---|
| Personal small project | No | Optional |
| Team project (with write access) | No | **Required** |
| Open-source contribution (no write access) | **Required** | **Required** |

**Team golden rule**: `main` must always be deployable. All new work happens on feature branches.

---

## 5. Two Main Workflows

### Workflow A: Your Own / Team Project (You Have Write Access)

```bash
git clone <url>                 # First time
cd <repo>
git pull                         # Every morning: sync first
git switch -c feature/xxx        # Create new branch
# ... edit code ...
git status                       # Check current state
git diff                         # See exact changes
git add .                        # Stage changes
git commit -m "describe change"  # Commit locally
git push -u origin feature/xxx   # First push needs -u
# Then open a PR on GitHub
```

### Workflow B: Open-Source Contribution (No Write Access)

```bash
# 1. Click "Fork" on GitHub (copies repo to your account)

# 2. Clone your fork
git clone https://github.com/yourname/project.git
cd project

# 3. Create a branch
git switch -c fix/typo-in-readme

# 4. Edit → add → commit → push
git add .
git commit -m "Fix typo in installation guide"
git push -u origin fix/typo-in-readme

# 5. Open "Compare & pull request" on GitHub
# 6. Wait for review and merge
```

**Why fork?** Because you don't have write access to the original repo. Forking to your own account is the way around permissions.

---

## 6. Command Cheat Sheet (by Frequency)

### Tier 1: Use 10× a day (muscle memory)
```bash
git status                   # Check state (zero risk, use constantly)
git add .                    # Stage all changes
git commit -m "msg"          # Commit
git push                     # Push to remote
git pull                     # Fetch + merge from remote
```

### Tier 2: A few times a week
```bash
git switch <branch>          # Switch branch
git switch -c <branch>       # Create and switch branch
git log --oneline            # Concise history
git diff                     # Show exact changes
git clone <url>              # Clone a repo (once per repo, for life)
```

### Tier 3: Occasional lifesavers
```bash
git restore <file>           # Discard working directory changes
git restore .                # Discard all uncommitted changes
git reset --soft HEAD~1      # Undo last commit (unpushed)
git revert <commit-id>       # Undo a pushed commit (adds new commit)
git stash                    # Temporarily shelve changes
git stash pop                # Restore shelved changes
git merge <branch>           # Merge a branch
```

### One-time config
```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

---

## 7. Scenario Playbook

| Situation | Command |
|---|---|
| First time encountering a project | `git clone` |
| Start of every workday | `git pull` |
| Starting a new task | `git switch -c feature/xxx` |
| Don't know current state | `git status` |
| Want to see what changed | `git diff` |
| Save progress (two steps) | `git add .` → `git commit -m "..."` |
| Share with others / back up | `git push` |
| Switch tasks | `git switch <branch>` (stash first if needed) |
| Review history | `git log --oneline` |
| Messed up, want to undo (uncommitted) | `git restore .` |
| Just committed, want to undo (unpushed) | `git reset --soft HEAD~1` |
| Undo a pushed commit | `git revert <commit-id>` |
| Submit code to team | Push, then open a PR on GitHub |

---

## 8. Common Confusions Clarified

### Q1: Do I need to clone a repo repeatedly?
**No.** Clone is the initial download. After that, use `git pull` for incremental sync. One clone per repo, for life (unless you switch computers).

### Q2: Won't the Git repo grow huge over time?
**No.** Git stores only deltas (diffs), not full copies + automatic compression + content deduplication. A 5-year small project's `.git` folder is typically just tens of MB.

**The only thing that bloats Git: accidentally committing large files.** You must use `.gitignore` to block them.

### Q3: Can multiple commits on the same branch cause conflicts?
**No.** Conflicts only happen during "merges" (when two parallel branches collide). Linear work is pure accumulation — never conflicts.

### Q4: Can I commit offline?
**Yes.** Commit is purely local; only push needs the network. You can commit freely on a plane, then push all at once when you land.

---

## 9. AI Engineer–Specific Notes

### Write a Good `.gitignore`

Never commit:
```
# .gitignore
__pycache__/
*.pyc
.ipynb_checkpoints/
venv/
env/
.env                    # ← NEVER push API keys!
data/                   # datasets
models/                 # model weights
*.pt
*.ckpt
*.bin
*.parquet
node_modules/
```

**Real incident**: Pushing `.env` with an OpenAI API key to a public repo → bots scrape it within minutes and rack up massive charges.

### 5 Leverage Points of Git for AI Iteration

1. **Precise diffs**: Know exactly what you changed
2. **Parallel branches**: Explore multiple experiments simultaneously
3. **Time machine**: Jump to any historical state instantly
4. **Commits as experiment logs**: e.g., `"Add dropout 0.3, acc=0.84 (+0.02)"`
5. **Psychological safety**: Dare to try bold changes without fear

### Commit Message Conventions

Good ✅:
```
Add user profile features with age and location
Fix NaN handling in tokenizer
Update learning rate to 3e-5, acc=0.87 (+0.02)
```

Bad ❌: `update`, `fix`, `xxx`, `asdf`

---

## 10. Work vs Personal Account Boundaries

### Red Lines (NEVER do these)
- ❌ Push company code to your personal GitHub (even a private repo)

### What to Put on Your Personal Account (Zero Compliance Risk)

1. **Learning notes / tech blog** (most recommended)
2. **Dotfiles / config** (30 minutes, one-time setup)
3. **Small tools / scripts** (things you've written casually)
4. **Tutorial follow-along projects** (learning outputs)
5. **Open-source contributions** (even a typo fix counts)
6. **Forks + Stars** (zero cost, shows activity)

---

## 11. 6 Modes for Learning via GitHub

### Mode 1: Read Source Code to Learn Architecture
Pick a library you use daily (`transformers`, `fastapi`), start from a function you know. **Press `.` on any repo page to open a VS Code editor in the browser.**

### Mode 2: Follow Pull Requests to Learn "Engineering Evolution"
Seeing how people **change** code is more valuable than seeing final versions. Reading a PR = watching the thought process of solving a real problem.

### Mode 3: Read Issues to Understand Real Problems
Search keywords you care about, read high-discussion issues. Learn real-world pitfalls and tradeoffs.

### Mode 4: Systematic Exploration via "Awesome" Lists
Search `awesome-llm`, `awesome-machine-learning` — maps of entire fields.

### Mode 5: Clone, Run, and Modify
`nanoGPT` is a must-do for AI engineers. Reading understanding < running understanding < modifying understanding.

### Mode 6: Follow Experts, Watch Their Activity Feed
In AI: `karpathy`, `huggingface`, `ggerganov`, `anthropics`, `openai`, and others.

### GitHub Keyboard Shortcuts (Game-Changers)

| Key | Action |
|---|---|
| `.` | **Open the repo in browser VS Code** (most magical) |
| `t` | File finder |
| `/` | Code search |
| `b` | View blame (who wrote each line) |
| `?` | Show all shortcuts |

---

## 12. 30-Day Kickoff Plan

### Week 1: Foundation
- Install Git on personal computer, configure `user.email`
- Register a personal GitHub account
- Create a `learning-notes` repo, write a README
- Run through a full clone → commit → push cycle

### Week 2: Add a Hands-on Repo
- Pick a recent topic you're learning, build a small demo
- Write a clear README

### Week 3: Set Up Dotfiles
- Put shell and VS Code configs in a `dotfiles` repo

### Week 4: Try an Open-Source Contribution
- Find a `good first issue`
- Even a typo fix counts — go through the full PR process

---

## The Core Philosophy

**Git's philosophy: History only moves forward. It never erases.**

**What GitHub is worth to an AI engineer**:
- ✅ Cloud backup (your code can't be lost)
- ✅ Rapid iteration tool (5 leverage points)
- ✅ The world's largest free coding school (read world-class source code)
- ✅ Technical identity card (essential for job hunting and interviews)

> You don't become a better writer by writing 1,000 words a day.
> You become better by **reading great works + writing some + reflecting**.
> GitHub gives you unlimited supply of the first.

---

*This is my first GitHub learning note — marking the beginning.*