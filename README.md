# Project Progress Orchestrator (PPO)

> A skill for keeping agents working toward a goal — without constant supervision.

---

## What This Is

```yaml
name: project-progress-orchestrator
description: Keep active projects moving with autonomous work cycles. When paired with scheduled crons and your agent runtime, this becomes a loop that orients, builds, verifies, and persists — staying silent unless there's real signal.
```

**The problem:** You set up an agent to work on a project. It does one impressive thing, then stops. Or it loses track of what it was doing. Or it floods you with updates about nothing.

**What this does:** PPO is a structured workflow that keeps an agent working toward a goal across hours, days, or weeks. It handles the boring but critical stuff: checking state before building, picking one concrete chunk, verifying the work, and persisting what happened. When paired with scheduled crons and your agent runtime, this is basically a loop.

**What makes it different:** The skill tightens itself. After each cycle, it evaluates what worked and what didn't, then adjusts its own procedures based on actual failure patterns. It molds to your codebase, your team, and your delivery cadence — not generic best practices.

---

## The Story

I built this for R2 — my AI assistant. I needed something that could make real progress on projects without me watching every step. The agent needed to:

- Wake up and understand where a project stood
- Clean up stale state before building
- Pick one concrete, unblocked chunk
- Build it, verify it, review it
- Persist what happened
- Stay quiet unless there was something I actually needed to know

PPO emerged from that need. It has run thousands of cycles since early 2026. This is the pattern I use to keep projects moving.

---

## The Core Loop

```
┌─────────────────────────────────────────────────────────────────┐
│  ORIENT  →  CLEAN  →  CHOOSE  →  BUILD  →  VERIFY  →  REVIEW   │
│     ↑                                                            │
│     └────────────────  PERSIST  ←  COMMUNICATE  ←───────────────┘
└─────────────────────────────────────────────────────────────────┘
```

1. **Orient** — Read the project contract, check for pause signals, understand current state
2. **Clean** — Fix stale claims, reconcile contradictions, settle recent work
3. **Choose** — Pick one concrete, unblocked chunk (respecting reviewer findings)
4. **Build** — Execute with a time budget, avoiding overlapping work
5. **Verify** — Run gates: lint, build, test, inspection
6. **Review** — Check output quality, catch errors before persistence
7. **Persist** — Write durable state: what changed, what's next, cycle-complete marker
8. **Communicate** — Silent by default; signal only when human action needed

---

## The Skill

See [`SKILL.md`](SKILL.md) for the complete specification including:
- Cron lifecycle management (start, stop, pause, status)
- Complete 10-step core loop with detailed instructions
- Reviewer Finding Contract (enforceable state, not prose)
- PM Escalation Template for human-in-the-loop decisions
- Digest patterns for twice-daily summaries
- Delivery wiring requirements

---

## The Diagram

![PPO Explainer Diagram](assets/ppo-diagram.jpg)

*An instructional diagram showing the PPO actor-critic style policy loop — note: this diagram illustrates the reinforcement learning-inspired conceptual model, not the operational workflow.*

---

## From PPO to Praxis

This skill is the foundation of **Praxis** — a harness for agent teams that we're building at DrC.ai.

While PPO defines the loop for a single project, Praxis extends it to:
- **Typed role contracts** — builder, reviewer, digest, auditor, improver
- **Explicit leases** — preventing overlapping work across agents
- **Run ledger** — durable, observable state for every cycle
- **Evaluator contracts** — machine-checkable outputs instead of prose claims
- **Meta-harness layering** — coordination across multiple projects

PPO proved the internal loop works. Praxis turns that pattern into a product.

---

## Who Built This

**Dr. Justin Crosby** — a physician and systems thinker exploring how autonomous agents can actually get things done. This is my first public open-source release. DrC.ai is where I share what I'm building.

---

## Usage

PPO is designed to run inside [OpenClaw](https://github.com/openclaw/openclaw) or compatible agent runtimes via scheduled cron jobs:

- **Builder loop** — every 30-60 min: cleanup-first, one chunk, verify, persist
- **Quality reviewer** — evaluates recent cycles for measurable progress
- **Twice-daily digest** — 9 AM / 9 PM summaries for human visibility

### Getting Started

This repo includes a **generic version** (`SKILL.md`) with placeholders for easy adaptation:

1. Read `SETUP.md` for customization instructions
2. Replace placeholders (`<USER_NAME>`, `<PM_TOPIC_ID>`, etc.) with your values
3. The skill works with any runtime that can trigger the 10-step loop and handle messaging

---

## License

MIT — Use it, fork it, build on it. If you find it useful, let us know what you're building.

---

## Connect

- DrC.ai — [drc.ai](https://drc.ai)
- Built with R2 — the agent that runs on this loop every day
