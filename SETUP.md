# Setup Guide

This skill is designed to be adapted to your specific agent setup and communication channels.

## Quick Start

1. **Copy `SKILL.md`** into your agent's skills directory
2. **Replace placeholders** with your actual values:
   - `<USER_NAME>` → your name or "the team"
   - `<PM_TOPIC_ID>` → your project manager/escalation channel ID
   - `<PROJECT_TOPIC_ID>` → your project discussion channel ID
   - `<HOST_MACHINE>` → your server/computer name (optional)
   - `<AGENT_USER>` → your agent's username (optional)

3. **Customize the cron payload** for your runtime (OpenClaw, custom, etc.)

## Placeholder Reference

| Placeholder | What to put | Example |
|-------------|-------------|---------|
| `the user` / `the owner` | Your name or role | "Sarah", "the team lead" |
| `<PM_TOPIC_ID>` | Where blockers/approvals go | `telegram:-1001234567890:topic:12` |
| `<PROJECT_TOPIC_ID>` | Where routine updates go | `telegram:-1001234567890:topic:5` |
| `<HOST_MACHINE>` | Your server name | "Ubuntu server", "Mac Studio" |
| `<AGENT_USER>` | Agent's system user | "agent", "bot" |

## Adapting for Your Runtime

This skill was originally built for **OpenClaw** with:
- Cron-scheduled `agentTurn` payloads
- Telegram delivery
- Isolated persistent sessions

If you use a different runtime (LangChain, AutoGPT, custom):
- The **Core Loop** (10 steps) remains the same
- Adapt the **Cron Design Pattern** section to your scheduler
- Replace **Delivery wiring** with your messaging system

## Minimal Viable Setup

The simplest version needs only:
1. A way to trigger the 10-step loop (cron, webhook, or manual)
2. A project status file to read/write
3. A way to message the user (for blockers/summaries)

## Original Reference

See `SKILL-ORIGINAL.md` for the author's original version with their specific configuration (R2 agent, Justin's setup, etc.).