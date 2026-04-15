# Teaching Claude Code to Remember
### A 5-Tier Persistent Context Architecture for Solution Engineers

**Author:** John Tehrani, Principal SE | **Built during:** Meridian Platform (28 days, 199 commits)
**Full interactive write-up:** https://jtehrani84.github.io/claude-context-architecture/

> 📎 **INSERT IMAGE:** `screenshots/hero-section.png`

---

## The Problem

Out of the box, Claude Code starts every session from scratch. No matter how many hours you've spent together, tomorrow it won't remember a thing.

**Context Window Bloat** — A monolithic CLAUDE.md that dumps 15,000 tokens every session. Deployment details when you're writing emails. Slide rendering rules when you're debugging Apex.

**Groundhog Day** — "I'm a Solution Engineer working on Salesforce. Here's my project structure. Don't use these words. Remember the 70/30 rule..." Every. Single. Session. The first 10 minutes are always re-orientation.

**No Compound Learning** — Claude makes the same mistake twice. Uses a deprecated product name you corrected last week. Suggests a pattern you've already rejected. Knowledge from yesterday's session vanishes overnight.

The real cost isn't time. It's that Claude can never get better at working with *you specifically*. Without persistent context, every session is a first date with a genius who has amnesia.

---

## The Architecture — Five Tiers

Instead of one massive instruction file, split context into five tiers. Load what you need, when you need it.

### P1: Identity — CLAUDE.md (Always Loaded | ~2,800 tokens)
Who you are, essential rules (anti-slop, anti-hallucination), and a routing table that tells Claude where to look things up. NOT a 1,000-line instruction dump. The irreducible minimum.

### P2: Session Init — Hooks (Auto on Start | ~200 tokens)
A Python script that fires on session start. Analyzes your working directory, git history, and branch name — then suggests which wiki pages to load. Context routing without manual intervention.

### P3: Knowledge Base — Wiki (On Demand | Variable)
A `wiki/` directory in your repo with deep reference pages organized by topic. Architecture patterns, deployment runbooks, design systems. Loaded only when the task needs them. Mine has 51 pages after a few months.

### P4: Long-Term Memory — Files (Index Loaded | ~2,000 tokens)
128 topic files holding decisions, corrections, feedback, and project state that persists across sessions. The MEMORY.md index is auto-loaded every session. Individual files are read on demand.

### P5: Behavioral Rules — Guardrails (Always Active | 10 files)
Standards you want enforced everywhere: security governance, architecture patterns, testing quality, communication style. Loaded from `~/.claude/rules/` every session. Not project-specific — these apply to everything.

> 📎 **INSERT IMAGE:** `screenshots/five-tier-architecture.png`

**The key insight:** Don't put everything in CLAUDE.md. Put a routing table there and detail in the wiki. Load what you need, when you need it.

---

## Before vs. After

| | Before | After |
|---|---|---|
| **Context at session start** | ~15,000 tokens (7.5%) | ~5,000 tokens (2.5%) |
| **Ramp-up time** | 10 minutes of re-orientation | Instant — hooks auto-route context |
| **Corrections** | Lost overnight | Persisted in memory files |
| **Quality consistency** | Varies session to session | Compound learning across sessions |
| **Context available for work** | 92.5% | 97.5% |

**3x less overhead.** The remaining 97.5% of your context window is available for actual work — code, conversation, and output.

> 📎 **INSERT IMAGE:** `screenshots/before-architecture.png` *(Before — monolithic CLAUDE.md)*
> 📎 **INSERT IMAGE:** `screenshots/after-architecture.png` *(After — 5-tier waterfall)*
> 📎 **INSERT IMAGE:** `screenshots/token-comparison.png` *(15K vs 5K token bars)*

---

## How I Got Here

This wasn't designed as 5 tiers on a whiteboard. Each tier emerged from a specific wall I hit during a 28-day platform build.

Started with everything in one CLAUDE.md. 15K tokens loading identically whether I was debugging a pipeline connector or drafting a customer deliverable. Corrections kept getting lost between sessions — things like "clasp push doesn't deploy, you also need clasp deploy" or "JWT audience is login.salesforce.com, not My Domain URL." Small, critical, and gone the next morning.

The question that forced the architecture: **what does Claude always need to know vs. what can it look up?** Once I framed it that way, each tier fell out of a different type of knowledge I was losing. Reference docs load differently than corrections. Behavioral standards apply everywhere. And the manual overhead of choosing what to load each session burned the first 10 minutes — so I had Opus automate it with a hook that reads git state.

**Five tiers from five types of knowledge loss.**

---

## The Karpathy Connection

This implements Andrej Karpathy's LLM Wiki pattern. Three layers:

- **Raw Sources** — articles, feed posts, docs. You curate what goes in.
- **Wiki** — Claude synthesizes, cross-references, and maintains the pages. You never organize it manually.
- **Schema** — CLAUDE.md + memory + rules. The structure that tells Claude how to use the wiki.

The maintenance burden is what kills every knowledge system. Claude handles all the bookkeeping — summarizing, cross-referencing, filing, consistency checking. Your job is sourcing, directing, and thinking. The wiki grows while you work.

### Where we went further than the pattern:

**Token economics** — Karpathy doesn't address context window costs. We engineered tiered loading: 2,800 tokens always-on, everything else on demand. 82% reduction vs. dumping everything into CLAUDE.md.

**Automated routing** — Karpathy's workflow is manual (you tell the LLM what to do). Our session-init hook analyzes your git state and auto-suggests which wiki pages to load. No human in the loop for context selection.

**Multi-modal persistence** — Karpathy has one artifact: the wiki. We separate four types — wiki (reference knowledge), memory (decisions and corrections), rules (behavioral standards), schema (identity and routing) — each loaded at different times for different reasons.

**Curation and lifecycle** — Karpathy describes Lint (health checks). We go further: a `/curate` command that scans memory for staleness, processes the wiki inbox, checks index health, and flags what needs attention. Memory files have decay rates — session logs go stale in 2 weeks, project state in a month, feedback and decisions almost never. His system grows but doesn't prune. Ours does.

> *"I think there is room here for an incredible new product instead of a hacky collection of scripts."*
> — Andrej Karpathy, on the LLM Wiki pattern

We took that seriously.

> 📎 **INSERT IMAGE:** `screenshots/karpathy-pattern.png` *(Three-layer mapping)*

---

## What About Maintenance?

The question everyone asks: doesn't the wiki and memory get stale over time? Yes — if you don't curate.

I run a `/curate` command once a week. Takes 5 minutes. Claude does all the legwork — surfaces candidates, shows a one-line summary, asks keep/update/delete. You're the judge, not the janitor.

Memory files have decay rates:
- **Session logs** — stale after 2 weeks
- **Project state** — stale after 1 month
- **Feedback and decisions** — almost never stale
- **Reference** — flag only if explicitly outdated

---

## Build Your Own in Stages

You don't need the full system on day one. Start with CLAUDE.md, add layers as you go. Each stage delivers value on its own.

### Stage 1: Foundation (30 minutes)
- Create a slim CLAUDE.md with identity, key rules, and a routing table
- Add 2-3 rules files in `~/.claude/rules/`
- Start a `wiki/` directory with an `index.md`

### Stage 2: Knowledge (ongoing)
- Add wiki pages as you work — Claude writes them, you curate
- Set up auto-memory with topic files
- Process your wiki inbox periodically

### Stage 3: Automation (1-2 hours)
- Add a SessionStart hook for context routing
- Set up `/curate` for weekly maintenance
- Configure decay rates for different memory types

### Stage 4: Curation (5 min/week)
- Run `/curate` weekly — Claude scans, you decide
- Archive old session logs, keep decisions and feedback
- Monitor MEMORY.md line count

---

## This Is Part 1 of a Series

This canvas covers the persistent context architecture — how to make Claude Code remember across sessions. More coming:

- **Part 1 (this one):** Memory, Wiki, CLAUDE.md, Hooks, Rules — the foundation
- **Part 2:** MCP Servers — extending what Claude can do (GitHub, search, browser, CRM data)
- **Part 3:** Custom Commands — SE-specific workflows (/account-prep, /deal-strategy, /deploy)
- **Part 4:** Plugins — context-mode (3-5x longer sessions), frontend-design, session analytics
- **Part 5:** Intelligence Pipeline — automated OSINT from X feed + web search

Each builds on the last. Start with Part 1, get value immediately, add the rest over time.

---

**Full interactive write-up:** https://jtehrani84.github.io/claude-context-architecture/

**Questions?** Drop them in the thread. Happy to do a 15-min walkthrough for anyone interested.
