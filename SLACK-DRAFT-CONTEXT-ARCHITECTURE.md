# Slack Draft — Claude Code Context Architecture Post

## Channel Message (short — drives to the link)

**I built a persistent context system for Claude Code and it cut my context overhead by 82%.** Here's the full write-up.

Claude Code starts every session from scratch — no memory of your projects, your preferences, the correction you made yesterday. I set up a 5-layer architecture (CLAUDE.md + wiki + memory + hooks + rules) that fixes this. Based on Karpathy's LLM Wiki pattern.

Full breakdown with architecture diagrams, before/after, and a starter template: https://jtehrani84.github.io/claude-context-architecture/

30 minutes to set up the basics. Questions in thread.

---

## Thread Reply 1 (the five layers)

Quick summary of the architecture:

1. **CLAUDE.md** — slim identity file (~180 lines) with a routing table that tells Claude where to look things up. NOT a 1,000-line instruction dump.

2. **Wiki** — a `wiki/` directory in your repo. Deep reference pages organized by topic. Loaded on demand, not upfront. Mine has 51 pages after a few months.

3. **Memory** — Claude's built-in auto-memory, structured with topic files. Corrections, decisions, project state that persists across sessions. 128 files and growing.

4. **Hooks** — Python scripts that fire on session events. The big one: a SessionStart hook that analyzes your working directory and suggests which wiki pages to load. Context routing.

5. **Rules** — `~/.claude/rules/` files for standards you want enforced everywhere. Security, architecture, testing, communication style.

The key insight: don't put everything in CLAUDE.md. Put a routing table there and detail in the wiki. Load what you need, when you need it.

---

## Thread Reply 2 (the Karpathy connection)

This implements Karpathy's LLM Wiki pattern from earlier this year. Three layers:

- **Raw Sources** — articles, feed posts, docs. You curate what goes in.
- **Wiki** — Claude synthesizes, cross-references, and maintains the pages. You never organize it manually.
- **Schema** — CLAUDE.md + memory + rules. The structure that tells Claude how to use the wiki.

The maintenance burden is what kills every knowledge system. Claude handles all the bookkeeping. Your job is sourcing, directing, and thinking. The wiki grows while you work.

Where we went further than the pattern:

1. **Token economics** — Karpathy doesn't address context window costs. We engineered tiered loading: 2,800 tokens always-on, everything else on demand. 82% reduction vs. dumping everything into CLAUDE.md.

2. **Automated routing** — Karpathy's workflow is manual (you tell the LLM what to do). Our session-init hook analyzes your git state and auto-suggests which wiki pages to load. No human in the loop for context selection.

3. **Multi-modal persistence** — Karpathy has one artifact: the wiki. We separate four types — wiki (reference knowledge), memory (decisions and corrections), rules (behavioral standards), schema (identity and routing) — each loaded at different times.

Karpathy wrote: *"I think there is room here for an incredible new product instead of a hacky collection of scripts."* We took that seriously.

---

## Thread Reply 3 (what about maintenance?)

The question everyone asks: doesn't the wiki and memory get stale over time? Yes — if you don't curate.

I run a `/curate` command once a week. Takes 5 minutes. It scans memory files for staleness, processes the wiki inbox, checks the index health, and flags anything that needs attention. Claude does all the legwork — surfaces candidates, shows a one-line summary, asks keep/update/delete. You're the judge, not the janitor.

Memory files have decay rates. Session logs go stale in 2 weeks. Project state goes stale in a month. Feedback and decisions almost never go stale. The curate command knows the difference and flags accordingly.

This is the 4th extension beyond Karpathy's pattern. He describes Lint (health checks). We go further: curation, archiving, staleness scoring. His system grows but doesn't prune. Ours does.

---

## Thread Reply 4 (this is part 1 of a series)

This write-up covers the persistent context architecture — how to make Claude Code remember across sessions. It's part 1 of a series:

- **Part 1 (this one):** Memory, Wiki, CLAUDE.md, Hooks, Rules — the foundation
- **Part 2:** MCP Servers — extending what Claude can do (GitHub, search, browser, CRM data)
- **Part 3:** Custom Commands — SE-specific workflows (/account-prep, /deal-strategy, /deploy)
- **Part 4:** Plugins — context-mode (3-5x longer sessions), frontend-design, session analytics
- **Part 5:** Intelligence Pipeline — automated OSINT from X feed + web search

Each builds on the last. Start with Part 1, get value immediately, add the rest over time.

---

## Slack Images (attach to channel message)

Best images for maximum visual impact in the Slack post:

1. **Hero shot** (attach to channel message): `screenshots/hero-section.png`
   - "Teaching Claude Code to Remember" with the 4 metric counters
2. **Five-tier architecture** (thread reply 1): `screenshots/five-tier-architecture.png`
   - P1-P5 gradient bars — the money shot that explains the system at a glance
3. **Before diagram** (thread reply 1): `screenshots/before-architecture.png`
   - Red monolithic CLAUDE.md block — shows the pain
4. **After diagram** (thread reply 1): `screenshots/after-architecture.png`
   - Cyan 5-tier waterfall — shows the solution
5. **Token comparison** (thread reply 1): `screenshots/token-comparison.png`
   - Red vs cyan bars — 15K vs 5K tokens, visceral comparison
6. **Karpathy pattern** (thread reply 2): `screenshots/karpathy-pattern.png`
   - Three-layer Karpathy mapping — adds credibility

All screenshots at: https://github.com/jtehrani84/claude-context-architecture/tree/main/screenshots

---

## Notes

- Live URL: https://jtehrani84.github.io/claude-context-architecture/
- Post in: #claude-code or relevant SE channel
- Timing: weekday morning, not Friday
- Follow-up: offer to do a 15-min walkthrough for anyone interested
- Series format builds anticipation and gives people time to implement before the next drop
