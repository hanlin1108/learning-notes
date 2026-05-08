# Claude Skills — Learning Note

## 1. Why do Skills exist?

LLMs are great at general tasks but **don't know your specific workflow** — your company's brand guidelines, your team's code style, the hidden gotchas of some internal tool. Re-explaining every time wastes tokens and produces inconsistent results.

Skills package that "expert knowledge" into folders that load **on demand**: only enter context when relevant, save tokens, give consistent results.

In one sentence: **an onboarding handbook for your agent.**

---

## 2. What is a Skill?

A folder containing a `SKILL.md` file:

```
my-skill/
├── SKILL.md          # Required: metadata + instructions
├── scripts/          # Optional: executable code
├── references/       # Optional: detailed docs (loaded on demand)
└── assets/           # Optional: templates, fonts, images
```

`SKILL.md` structure:

```markdown
---
name: pdf-form-filler
description: Fills out PDF forms. Use when the user mentions "fill a form",
  "PDF form", "submit application", or similar.
---

# Instructions (loaded only after the skill triggers)
1. Use scripts/extract_fields.py to extract fields
2. Fill in based on user input
3. ...
```

**Progressive disclosure** — the core design:

| Level | Content | When loaded |
|-------|---------|-------------|
| L1 | name + description | Always in the system prompt |
| L2 | SKILL.md body | When description matches the task |
| L3 | references/ files, scripts/ | Only when Claude decides it needs them |

So you can install dozens of skills with almost no token overhead at idle.

---

## 3. How to develop a Skill?

**Fastest path: use the official `skill-creator` skill** to draft one for you.

By hand, three steps:

**Step 1. Get clear on intent**

- What should this skill let Claude do?
- When should it trigger? What might the user say?
- What's the output format?

**Step 2. Write SKILL.md**

```markdown
---
name: brand-deck
description: Generates slide decks following our brand guide. Use whenever
  the user asks for slides, a deck, a presentation, a proposal, or a
  briefing — even if they don't say "brand".
---

## Must-follow rules
- Font: Inter
- Primary color: #0A2540
- Cover slide: templates/cover.pptx

## Workflow
1. Read references/brand-guide.md
2. ...
```

**Step 3. Key principles**

- **Make `description` "pushy"** — Claude tends to *under*-trigger by default. List multiple phrasings users might use ("slides", "deck", "presentation", "briefing").
- **Keep the body lean** — once loaded, it stays in context every turn. Every line is a recurring token cost. Say *what to do*, not long explanations of *why*.
- **Push details to sub-files** — keep SKILL.md under ~500 lines. Long content goes in `references/` so Claude reads it only when needed.
- **Use scripts for deterministic logic** — sorting, parsing, validation, etc. should be Python scripts Claude executes. More reliable than asking the model to do it, and the script source never enters context.

---

## 4. How to use Skills?

Three environments, same behavior:

**Claude.ai (web/mobile)**

1. Settings → Customize → Skills
2. Toggle on the skills you want (Code execution must be enabled first)
3. Upload custom skills (zip the folder)
4. Triggering is **automatic** — Claude uses a skill when the task matches its description; no `@`-mention needed

**Claude Code**

- Personal: drop into `~/.claude/skills/my-skill/SKILL.md`
- Project: drop into `.claude/skills/`, commit alongside code
- Hot-reloads as you edit during a session

**API**

- Loaded via the code execution tool — see the Skills API Quickstart
- ⚠️ Skills do **not** sync between Claude.ai, Claude Code, and the API. Upload separately on each surface.

---

## 5. How to optimize and iterate?

**Core loop: write → test → review → revise**

1. **Build a test set** — 5–10 representative prompts
2. **Run baseline** (skill off) and **with-skill** (skill on)
3. **Watch two things**:
   - **Trigger rate** — does it fire when it should? If not, make the description more "pushy" with more user phrasings
   - **Output quality** — when it does fire, is the result right? If not, fix the body
4. **Quantify what you can** — assertions for objective checks (file format, required fields). Subjective stuff (tone, design) needs human review
5. Run `skill-creator`'s built-in description optimizer to tune trigger accuracy

**Common issues:**

- Doesn't fire when it should → add more trigger phrases, make the language more direct
- Goes off the rails → add explicit "**never do this**" anti-examples
- Inconsistent output → freeze variable parts into a script

---

## 6. How to remove a Skill?

**Claude.ai:** Customize → Skills → find the skill → toggle off; to delete entirely, click `...` → Delete.

**Claude Code:** Just delete the `~/.claude/skills/skill-name/` folder. Gone next session.

**API:** Stop referencing it in the code execution container.

---

## 7. How to download Skills?

Main sources:

- **Anthropic's official repo**: `github.com/anthropics/skills` — many examples, clone and use or adapt
- **Anthropic built-ins** (already on Claude.ai, no download needed): docx, pptx, xlsx, pdf
- **Third-party marketplaces / community shares**: discover via agentskills.io

What you get is a folder. Zip it and upload to Claude.ai, or copy into `~/.claude/skills/`.

---

## 8. Common Skills

**Anthropic built-ins (ready to use):**

| Skill | Purpose |
|-------|---------|
| `docx` | Create/edit Word docs (TOC, headers, tables) |
| `pptx` | Create/edit PowerPoint |
| `xlsx` | Excel with formulas and charts |
| `pdf` | PDF create, merge, split, fill, watermark |
| `frontend-design` | Generate frontend UI with real design quality, avoiding generic AI look |
| `skill-creator` | Helps you create and optimize other skills (meta-skill) |
| `pdf-reading` / `file-reading` | Efficient large-file reading |

**Typical custom-skill scenarios:**

- Company brand guide (colors, fonts, logo usage)
- Internal API conventions
- Code review checklist
- Per-client report templates
- Test case generation rules

---

## 9. Best practices & lessons from the field

**From Anthropic's internal use (hundreds of skills in production):**

1. **Start small** — most good skills started as a few lines. Add a rule each time you hit a new edge case.
2. **One skill, one job** — don't aim for an "all-in-one" skill. Split for cleaner triggering and easier maintenance.
3. **Evaluate before authoring** — find where Claude actually fails today, then build a skill to fix that gap. Don't write to imagined needs.
4. **Structure beats prose** — code blocks, lists, and tables are easier to follow than long paragraphs.
5. **Examples > explanations** — one good example + one bad example beats three paragraphs of theory.
6. **Be careful with frontmatter** — it lands in Claude's system prompt. Sloppy content there can pollute behavior.
7. **Mark "executable" vs "reference" code clearly** — should Claude *run* this script or just *read* it for reference? Spell it out.
8. **Monitor usage with hooks** — skills that rarely fire are either unneeded or have bad descriptions.
9. **Don't over-engineer** — most tasks need just one SKILL.md, not ten subdirectories.
10. **Version control everything** — a skill is a folder. Put it in git alongside your code.

**Pitfalls to avoid:**

- ❌ Vague description ("processes data") → never triggers
- ❌ Multi-thousand-line SKILL.md → fills context, slows everything down
- ❌ API keys or secrets in the skill → skills get shared, that's a leak
- ❌ Assuming skills sync across surfaces → they don't, upload everywhere
- ❌ Using the LLM for deterministic work → sorting, math, parsing belong in scripts

---

## 10. Full example: building a "Daily Brief" skill with skill-creator

Here's a real walkthrough — from prompt to working skill.

### Step 1. Trigger skill-creator

```
You: Help me build a skill that gives me a news brief every morning.
```

skill-creator loads and starts the interview.

### Step 2. Interview (4 key questions)

```
Claude: Before I write anything, four questions:
  1. What should this skill let me do?
  2. When should it trigger? How might you phrase the request?
  3. What does the output look like?
  4. What topics? Where does the data come from?

You:
  1. Search the last 24 hours of news, group by my interests, summarize
  2. Trigger on "daily brief", "morning brief", "what's the news today",
     "today's headlines"
  3. Categorized list, 1–2 sentences per item with link, 5-min read
  4. AI/tech, global finance, China internet, SaaS industry; web_search
```

### Step 3. skill-creator produces files

Folder layout:

```
daily-brief/
├── SKILL.md
└── references/
    └── preferences.md     # personal prefs in their own file —
                           # editable without touching the body
```

**SKILL.md:**

```markdown
---
name: daily-brief
description: Generates a personalized daily news brief. Use whenever the
  user says "daily brief", "morning brief", "what's the news today",
  "today's headlines", "what happened today", or asks vaguely about news
  in a way that suggests they want a daily roundup. Trigger even if the
  user doesn't explicitly say "brief".
---

# Daily Brief

## Step 1: Read preferences
Read `references/preferences.md` for topics, language, tone, length.

## Output format (follow strictly)

# Daily Brief · YYYY-MM-DD

## 🔥 Top stories (3)
- **One-line headline**. 1–2 sentences of context. [source]

## 🤖 AI / Tech (2–3)
- ...

## 💰 Finance (2–3)
- ...

## 👀 Watch today
- 1–2 things happening today (earnings, launches, data prints)

## Workflow

1. Call `user_time_v0` for the current time; define the past-24h window
2. Run **one** web_search per topic in preferences.md;
   include the current year in the query to filter out stale results
3. For top 1–2 headlines, use web_fetch to verify details —
   search snippets are often too thin
4. Drop: anything older than 24h, obvious duplicates, advertorial fluff
5. Output in the format above

## Never

- ❌ Long commentary or analysis (this is a brief, not an op-ed)
- ❌ Make up info that isn't in the search results
- ❌ Quote more than 15 words from any single source
- ❌ Put the same story in multiple categories

## Length

400–600 words, ~5-min read. On slow news days, prefer shorter over padding.
```

**references/preferences.md:**

```markdown
# Topics (in priority order)
- AI / LLM progress (OpenAI, Anthropic, Google, Meta, ByteDance, DeepSeek)
- Global finance (Fed, US equities, FX, oil)
- China internet / tech policy
- SaaS / enterprise software

# Output preferences
- Language: English
- Tone: restrained, high information density, no clickbait
- Length: 5-min read
```

### Step 4. Run the test set

skill-creator generates 4 test prompts:

| # | Prompt | What to check |
|---|--------|---------------|
| 1 | "Give me today's daily brief" | Standard trigger |
| 2 | "What's the news today?" | Vague trigger — does it catch on? |
| 3 | "Show me the latest US bank earnings" | **Should NOT trigger** — single-topic query |
| 4 | "Morning brief me" | Casual phrasing |

### Step 5. Find issues → iterate

After running:

- ✅ 1 and 4 fire correctly
- ❌ **2 didn't fire** — Claude judged "what's the news today?" too vague
- ✅ 3 didn't fire — correct

Fix: make the description pushier — explicitly include vague phrasings (the version above already reflects this). Re-run, 2 now fires.

Then notice headlines occasionally include news from 36 hours ago. Add a rule to step 4: **"verify publication time with web_fetch; drop anything older than 24h, no exceptions"**.

### Best practices this example uses

| Practice | Where it shows up |
|----------|-------------------|
| Pushy description | Lists 6 user phrasings + "trigger even if not explicitly stated" |
| Progressive disclosure | Preferences in `references/preferences.md`, body untouched when prefs change |
| Strict output format | Fixed template, not a tone description |
| Anti-pattern list | "Never" section with 4 explicit don'ts |
| Tools cover LLM weak spots | `user_time_v0` for time, `web_fetch` for verification — not guessing |
| Tests cover negatives | Prompt 3 is deliberately *not* supposed to trigger |

After this round the skill is usable. When you find new issues (say it triggered but skipped finance), come back and add another rule. **Skills grow over time** — don't try to nail it on the first draft.

---

## TL;DR

> A skill is a folder with a `SKILL.md`, feeding "expert knowledge" to Claude on demand.
>
> **Write**: name + a "loud" description + a lean body + optional scripts/references.
> **Use**: toggle in Claude.ai, drop into `.claude/skills/` for Code, upload via API.
> **Iterate**: run a test set, tune for trigger rate and output quality.
> **Iron rules**: one skill per job, brevity over completeness, deterministic work goes in scripts.

---

*Last updated: 2026-05-08*
