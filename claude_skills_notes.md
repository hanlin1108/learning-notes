# Claude Skills — Learning Note

> A **skill** is a folder of "expert knowledge" Claude loads on demand. Think of it as an onboarding handbook for your agent.

---

## Why

LLMs don't know your workflow — your brand guide, code style, internal tools. Re-explaining every chat wastes tokens and gives inconsistent results. Skills solve this: each one only enters context when relevant.

---

## What's in a Skill

```
my-skill/
├── SKILL.md          # required — metadata + instructions
├── scripts/          # optional — Python Claude executes
└── references/       # optional — long docs, loaded only when needed
```

**SKILL.md** has frontmatter (always loaded) + body (loaded when triggered):

```markdown
---
name: brand-deck
description: Generates slide decks following our brand guide. Use when
  the user asks for slides, a deck, a presentation, a proposal, or a
  briefing — even if they don't say "brand".
---

## Rules
- Font: Inter
- Primary color: #0A2540

## Workflow
1. Read references/brand-guide.md
2. ...
```

**Progressive disclosure** — the core idea:

| Level | Content | Loaded |
|-------|---------|--------|
| L1 | name + description | always |
| L2 | SKILL.md body | when description matches |
| L3 | references/, scripts/ | only if Claude needs them |

You can install dozens of skills with near-zero idle token cost.

---

## Authoring rules that matter

- **Make `description` pushy.** List multiple user phrasings ("slides", "deck", "presentation"). Claude *under*-triggers by default.
- **Keep the body lean.** Every line is a recurring token cost once loaded. Say *what to do*, not *why*.
- **Push detail to `references/`.** Aim for SKILL.md under ~500 lines.
- **Put deterministic work in scripts.** Sorting, parsing, validation — let Python do it. More reliable, and the source never enters context.
- **Add a "Never do" section.** Explicit anti-examples beat soft prose.

---

## How to use

| Surface | How |
|---------|-----|
| **Claude.ai** | Settings → Customize → Skills → toggle on or upload zip. Triggers automatically. |
| **Claude Code** | Drop folder into `~/.claude/skills/` (personal) or `.claude/skills/` (project). Hot-reloads. |
| **API** | Load via the code execution tool (see Skills API Quickstart). |

> ⚠️ Skills do **not** sync between surfaces. Upload separately on each.

**Fastest way to write one:** invoke the official `skill-creator` skill — it interviews you and drafts the files.

---

## Where to get skills

- **Anthropic built-ins** (already on Claude.ai, no install): `docx`, `pptx`, `xlsx`, `pdf`, `frontend-design`, `skill-creator`, `pdf-reading`
- **Official examples**: [github.com/anthropics/skills](https://github.com/anthropics/skills) — clone and adapt
- **Community**: agentskills.io and similar marketplaces

What you download is just a folder — zip it for Claude.ai, or drop into `~/.claude/skills/`.

---

## Iteration loop

1. **Test set** — 5–10 prompts (include negatives that should *not* trigger).
2. **Watch two metrics:**
   - **Trigger rate** — fires when expected? If not, make the description pushier.
   - **Output quality** — when it fires, is it right? If not, fix the body.
3. **Common fixes:**
   - Doesn't fire → add more trigger phrases
   - Goes off the rails → add explicit "Never" rules
   - Inconsistent output → move that step into a script

---

## Example: a "Daily Brief" skill

**Goal:** trigger on phrases like "morning brief" → search last 24h news → output a categorized summary.

**Workflow** (the body of SKILL.md):

1. Read `references/preferences.md` for topics + tone
2. Run one `web_search` per topic, with current year in the query
3. Use `web_fetch` on top headlines to verify dates
4. Drop anything >24h old or duplicate
5. Output in fixed template (Top stories / AI / Finance / Watch today)

**The lesson from iterating on it:**

| Test prompt | Outcome |
|-------------|---------|
| "Daily brief" | ✅ fires |
| "What's the news today?" | ❌ didn't fire — too vague |
| "Show me US bank earnings" | ✅ correctly didn't fire (single topic) |

Fix: add the vague phrasing explicitly to the description. **Skills grow by patching real failures**, not by trying to be perfect on draft one.

---

## Removing a skill

| Surface | How |
|---------|-----|
| **Claude.ai** | Customize → Skills → toggle off, or `...` → Delete |
| **Claude Code** | `rm -rf ~/.claude/skills/skill-name/` |
| **API** | Stop loading it in the code execution container |

---

## Pitfalls

- ❌ Vague description ("processes data") → never triggers
- ❌ Bloated SKILL.md → fills context every turn
- ❌ Secrets in the skill → skills get shared
- ❌ Asking the LLM to do deterministic work → put it in a script

---

## TL;DR

> A skill = folder + `SKILL.md`. Loud description, lean body, scripts for hard logic, references for long docs. Drop into `.claude/skills/` or upload to Claude.ai. Iterate on trigger rate and output quality. One skill, one job.

---

*Last updated: 2026-05-08*
