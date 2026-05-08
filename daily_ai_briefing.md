# Daily AI Briefing — Prompt

> A reusable prompt for turning any capable LLM into your personal AI intelligence analyst. Run it daily; read the output in 3 minutes over coffee.

---

## The idea

Most people stay informed about AI by subscribing to newsletters, following Twitter, checking Reddit — all pull-based. You have to remember to look, and you have to filter the noise yourself. This doesn't scale. The field moves too fast, the sources are too scattered, and the signal-to-noise ratio is terrible.

The idea here is different. The LLM acts as your personal intelligence analyst. Every day, it searches across the entire landscape — lab announcements, GitHub, key people, research, industry news — and delivers one short briefing that contains only what actually matters. You don't browse. You don't filter. You read the briefing in 3 minutes over coffee, and you're done.

The key insight: the LLM is better at this than you are. It can check many sources in parallel, cross-reference them, distinguish genuine breakthroughs from marketing fluff, and compress everything into a few paragraphs. The human's job is to read the output and decide what to act on. The LLM's job is everything else.

## What to monitor

Five dimensions, roughly in priority order:

1. **Releases** — new models, major product updates, API changes, pricing shifts from OpenAI, Anthropic, Google, Meta, Mistral, and others. This is the highest-signal category.
2. **Tools & open source** — new repos gaining traction, new developer tools, new workflows. The kind of thing that changes how you work. Think Karpathy's LLM Wiki, Cursor, Claude Code — paradigm-level shifts in tooling.
3. **Key people** — what Karpathy, Altman, Hassabis, LeCun, Simon Willison, Ethan Mollick, and a handful of others are saying and building. These people are often the primary source — news follows them, not the other way around.
4. **Industry** — major funding, acquisitions, policy changes, regulation. Only the stuff that actually changes the landscape.
5. **Research** — papers and techniques likely to hit production in 1 month. Ignore purely theoretical work unless it's exceptional.

The specific sources will shift over time. New labs will emerge, people will change roles, platforms will rise and fall. The agent should adapt. This list communicates intent, not a rigid checklist.

---

## Trigger prompt

Paste this into your LLM agent (Claude Code, OpenClaw, Codex, etc.) to run a daily briefing:

> send me the email with your report now, with top10 each one 1-3 sentences make it very concise, easy to understand and structured
