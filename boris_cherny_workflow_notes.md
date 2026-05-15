# Boris Cherny's Claude Code Workflow

> The Creator's Playbook — Learning Notes

---

## Who Is Boris Cherny?

Staff Engineer at Anthropic. Creator and head of Claude Code. In January 2026, he publicly shared his entire workflow on X, and the dev community went wild.

---

## The Big Idea

Don't code line by line. **Orchestrate multiple AI agents like a strategist.** One person can produce the output of a small engineering team.

---

## Table of Contents

1. [Run 5 Claudes in Parallel (Terminal)](#1-run-5-claudes-in-parallel-terminal)
2. [Run 5–10 More on the Web](#2-run-510-more-on-the-web)
3. [Use Opus + Thinking, Always](#3-use-opus--thinking-always)
4. [Keep a Shared CLAUDE.md (~100 lines)](#4-keep-a-shared-claudemd-100-lines)
5. [Tag Claude in Code Reviews](#5-tag-claude-in-code-reviews)
6. [Start Every Task in Plan Mode](#6-start-every-task-in-plan-mode)
7. [Slash Commands for Repetitive Workflows](#7-slash-commands-for-repetitive-workflows)
8. [Use Subagents for Specialized Tasks](#8-use-subagents-for-specialized-tasks)
9. [Post-Tool-Use Hooks](#9-post-tool-use-hooks)
10. [Test Every Change with the Chrome Extension](#10-test-every-change-with-the-chrome-extension)
11. [Throw Away 10–20% of Output](#11-throw-away-1020-of-output)
12. [Compound Your Learnings](#12-compound-your-learnings)
13. [Your Role = Strategist, Not Typist](#13-your-role--strategist-not-typist)
14. [The 3 Core Principles](#the-3-core-principles)
15. [Quick-Start Checklist](#quick-start-checklist)

---

## 1. Run 5 Claudes in Parallel (Terminal)

- Open 5 terminal tabs, each running its own Claude Code session
- Number tabs 1–5, use hotkeys (Cmd+1, Cmd+2…) to switch
- Each session works on a **separate git checkout** (not branches — full checkouts to avoid conflicts)
- Turn on **system notifications** so you know when a session needs input

## 2. Run 5–10 More on the Web

- Open additional sessions at `claude.ai/code` in your browser
- Use the **teleport** command to move sessions between local terminal ↔ web
- You can even start sessions from your **phone** (Claude iOS app) and check results later

## 3. Use Opus + Thinking, Always

- Boris uses the most capable model (Opus with extended thinking) for everything
- It's slower per-token but needs **less steering and fewer corrections**
- Net result: faster overall, especially when running parallel sessions
- Switch models mid-session with `/model`

## 4. Keep a Shared CLAUDE.md (~100 lines)

- A markdown file in your project root that Claude reads at the start of every session
- Boris's is only ~100 lines / ~2,500 tokens — every line earns its place
- **The golden rule:** when Claude does something wrong, add a rule to CLAUDE.md so it never repeats the mistake
- Check it into Git. The whole team contributes.
- Bootstrap one with `/init`

## 5. Tag Claude in Code Reviews

- During PR reviews, tag `@claude` to update CLAUDE.md as part of the PR
- Uses the Claude Code GitHub Action
- Feedback loop: human spots issue → Claude updates rules → future sessions avoid it automatically

## 6. Start Every Task in Plan Mode

- Press **Shift+Tab twice** to enter Plan mode
- Go back and forth with Claude until the plan is solid
- Then switch to auto-accept mode — Claude can usually one-shot the implementation
- Also great for: bug investigation, codebase deep dives, perf optimization

## 7. Slash Commands for Repetitive Workflows

- Create custom commands in `.claude/commands/` (checked into Git)
- Example: `/commit-push-pr` — handles git add, commit message, push, and PR creation in one step
- Boris runs this dozens of times daily
- To create one: just ask Claude Code to make it for you

## 8. Use Subagents for Specialized Tasks

- **Code Simplifier** — cleans up and simplifies code after the main work
- **Verify App** — runs end-to-end tests before shipping
- Subagents keep the main context window clean (one task per agent)
- For complex problems: throw more compute at it, not more time

## 9. Post-Tool-Use Hooks

- Auto-run formatters (like Prettier) after every file write
- Keeps code style consistent without Claude needing to think about formatting
- Configure in your Claude Code settings

## 10. Test Every Change with the Chrome Extension

- Claude opens a browser, tests the UI, and iterates until the code works and the UX feels good
- A different kind of E2E testing — you don't write the test, Claude just knows how to validate

## 11. Throw Away 10–20% of Output

- Not everything Claude produces is good. That's expected.
- Be willing to discard and re-run. It's cheaper than debugging bad code.

## 12. Compound Your Learnings

- Every mistake becomes a permanent lesson (via CLAUDE.md)
- Every workflow gets automated (via slash commands)
- Every pattern gets documented (via code review)
- The system gets smarter over time without extra effort

## 13. Your Role = Strategist, Not Typist

- You're directing, reviewing, and deciding — not writing syntax
- Think of it like playing StarCraft: commanding autonomous units
- Focus on code review and steering; when you read a PR, the code is already in good shape

---

## The 3 Core Principles

From Boris's CLAUDE.md:

| Principle | What It Means |
|---|---|
| **Simplicity** | Make every change as simple as possible. Delete lines instead of adding them. |
| **Root Causes** | No band-aids. Find and fix the real problem. Hold to senior-dev standards. |
| **Minimal Surface** | Only touch what's necessary. No side effects. Don't break working things. |

---

## Quick-Start Checklist

- [ ] Install Claude Code (terminal-based, runs via CLI)
- [ ] Run `/init` in your project to generate a starter CLAUDE.md
- [ ] Add 3–5 rules to CLAUDE.md based on mistakes you've seen
- [ ] Create your first slash command (start with `/commit-push-pr`)
- [ ] Try Plan mode (Shift+Tab ×2) before your next feature
- [ ] Open a second terminal tab and run two sessions in parallel
- [ ] After each code review, update CLAUDE.md with new learnings

---

*Source: Boris Cherny's X thread (Jan 4, 2026), VentureBeat, InfoQ, Push To Prod (Substack)*
