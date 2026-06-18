---
name: project-progress-orchestrator
description: Keep active projects moving cleanly with autonomous or manual work cycles: cleanup-first project continuation, scheduled cron/agentTurn design, stale source-of-truth reconciliation, checkpoint/memory updates, verification gates, approval-boundary handling, and silent-unless-signal progress loops. Use when asked to keep a project moving, design/update recurring project-builder crons, clean up after autonomous runs, continue all unblocked work, audit whether scheduled runs made real progress, or generalize a project-progress workflow into reusable automation.
---

# Project Progress Orchestrator

## Skill Lifecycle (start / stop / status)

The PPO crons ARE the skill at runtime. The skill loop only runs when its crons are enabled. Invoking the skill in chat without managing the crons does not create a sustained loop.

### Start / invoke
When Justin says "invoke PPO", "start PPO", "start the skill", "keep <project> moving", or equivalent for a project:

1. Check whether the project's PPO crons exist. Naming pattern: `<project> autonomous project builder`, `<project> quality reviewer`, `<project> daily digest` (or close variants).
2. If missing → create from `Cron Design Pattern`, `Standard Cron Prompt Template`, and `Digest Cron Prompt Template` below, with explicit delivery wiring by cron role: builder/digest announce to the project topic; reviewer delivery is `none`.
3. If present but disabled → re-enable. Do NOT delete and recreate; tuned config (delivery routing, scope, prompt fixes) must be preserved.
4. Force-run the builder once and confirm delivery lands in the expected topic.
5. Report to Justin: which crons are now enabled, next scheduled run, where output will land.

### Stop / pause
When Justin says "stop PPO", "pause the skill", "stop the loop", or equivalent for a project:

1. **Disable** the project's PPO crons (`enabled: false`). Do NOT delete — disable preserves per-project tuning so re-invoke is clean.
2. Update each disabled cron's description with `"DISABLED <YYYY-MM-DD HH:MM ET> by Justin: <short reason>"`.
3. Confirm to Justin which crons were disabled and that config is preserved for re-invoke.

Delete-and-recreate is only used when Justin explicitly says "retire PPO for <project>" or equivalent.

### Complete-block pause
When a project is completely blocked — no safe builder, reviewer, digest, cleanup, local documentation, approval-packet, or internal-only fallback work remains — pause the project's entire PPO set until Justin directs otherwise:

1. Disable all project PPO crons (`builder`, `reviewer`, `digest`, and any close variants). Do not delete them.
2. Update each cron description with `"PAUSED <YYYY-MM-DD HH:MM ET>: completely blocked pending Justin direction — <exact blocker>"`.
3. Persist the blocker in the project checkpoint/source of truth with the missing input or approval needed to resume.
4. Tell Justin once, in the project topic and/or PM topic per escalation rules, that the full project loop is paused and exactly what reference set/decision/approval unblocks it.
5. Do not re-enable automatically. Resume only after explicit Justin direction, new source material, or the missing approval/input arrives.

Do not use `NO_REPLY` to hide a complete-block pause; disabling the loop is material and must be surfaced once.

### Visualization / approval gate pause
Before any new directory, UI, prototype, dashboard, app surface, substantial template build, or material user-facing workflow build:

1. Produce an approval packet before implementation: structure diagram, flow diagram, UI mockup/wireframe, feature inventory, explicit out-of-scope list, acceptance gates, and cron restart plan.
2. Use Mermaid or Excalidraw based on whichever gives the clearest accurate representation; use higher-fidelity HTML/image mockups when UI feel or layout judgment matters.
3. Pause the applicable PPO crons (`builder`, `reviewer`, `digest`, and support scans/health monitors that would otherwise feed the blocked loop) while waiting for Justin approval. Disable, do not delete.
4. Update cron descriptions with `"PAUSED <YYYY-MM-DD HH:MM ET>: pending Justin visualization/approval gate — <packet/artifact path or missing approval>"`.
5. Persist the approval gate in the project source of truth/checkpoint, including the exact packet path, what approval unlocks, and which cron IDs should be re-enabled or rebuilt after approval.
6. Do not continue implementation or run empty retry loops while approval is pending. Allowed work is limited to improving the approval packet, reconciling docs, or fixing safe local issues that directly improve Justin's ability to decide.
7. After Justin explicitly approves the packet, reactivate preserved PPO crons or build successor crons from the approved scope, force-run the smallest safe builder cycle if appropriate, and report which jobs resumed and what they will do next.

### Runtime fault / restart guard

If PPO appears unhealthy because of runtime behavior outside a single project prompt — stale cron cache, wrong model after config update, repeated delivery resolver failure, stuck job state, or another condition that may require an OpenClaw gateway restart:

1. First preserve **all active PPO topic session state**, not just the affected project: confirm each active project checkpoint/source-of-truth, today's memory, and any declared `open_findings_file` contain the latest builder/reviewer/digest state.
2. Prefer a targeted fix first: correct the affected cron payload/delivery/prompt, force-run the smallest safe verification, and monitor the next cycle.
3. Restart the gateway only if the same issue recurs after the targeted fix and state preservation is complete.
4. If the issue is isolated to one PPO workflow and cannot be safely fixed without deeper investigation, pause only that affected workflow's PPO crons, preserve its blocker in the project source of truth/checkpoint, and notify once in that project's topic with the plain-English blocker and what will be needed to resume.
5. Do not leave a known-bad workflow running just to collect more evidence if it is creating noise, wrong-topic output, unsafe actions, or state drift.

### Milestone / work-package completion retirement

Recurring crons that are scoped to a finite milestone, sprint phase, work package, or named checklist must not keep running after their acceptance gates are satisfied.

When a builder, reviewer, or digest determines that its underlying work is complete:

1. Confirm completion from the source of truth and verification gates, not only from the prior chat message.
2. Persist the completion in the project checkpoint/source of truth with the completed milestone, verification, final commit/branch state if relevant, and the next milestone or explicit handoff.
3. If follow-on work exists, create a fresh cron set for the next milestone with new scope, acceptance gates, and delivery wiring before retiring the completed set. Never mutate a completed milestone cron into the next milestone unless Justin explicitly asks to preserve that exact job ID.
4. Disable every cron in the completed milestone set (`builder`, `reviewer`, `digest`, and close variants) immediately after the successor set exists, or immediately if no successor is needed.
5. Delete the completed milestone crons after the completion state is persisted and successor handling is resolved. Do not leave stale milestone crons enabled or merely paused where they can be accidentally re-enabled.
6. Tell Justin once in the project topic: completed milestone, retired cron names/IDs, and the next cron set/status.

Exception: standing project-level PPO loops that are meant to run indefinitely are disabled on pause/stop and preserved for re-invoke. Only finite milestone/work-package crons are retired and deleted on completion.

### Status
When Justin says "PPO status" or asks whether the skill is running:

1. List all PPO crons (builder / reviewer / digest) per project.
2. For each: enabled? last run timestamp (ET), last status, last delivery status, next scheduled run.
3. Note any project with `.r2-pause` file or non-expired `r2-pause-until:` in PROJECT.md.
4. Flag any non-reviewer cron with `delivery.mode: none`, any cron with `delivery.to: last`, or recent delivery resolver errors. Reviewer `delivery.mode: none` is expected.

### Manual single-cycle invoke
If Justin wants exactly one cycle without enabling the cron loop, say so explicitly: "run one PPO cycle for <project>, don't enable the cron." Default invoke = manage the crons, not just one-shot.

## Core Loop

Use this sequence for each project cycle, whether triggered manually or by cron:

1. **Orient from source of truth**
   - **Check for pause signal first.** If `.r2-pause` exists in the project folder, OR if `AGENTS.md` / `PROJECT.md` contains a non-expired `r2-pause-until: <ISO timestamp>` line, log "paused, skipping cycle" and exit immediately. Do not read further, do not reconcile, do not commit. Default reason for pause: human is editing in another tool (Claude Code, manual review, etc.).
   - **Read `reconcile_cadence`** from project `PROJECT.md` (default: `active` if absent). Apply per the table in Cron Design Pattern. If the cadence says skip this cycle, exit cleanly with `cadence: <value>, skipping` log line.
   - Read the project contract: `ACTIVE_LANES.md`, project `PROJECT.md`/`README.md`/`STATUS.md`, relevant checkpoint, then today's memory.
   - Prefer explicit project docs over stale memory; reconcile conflicts into the project file or checkpoint.

2. **Clean before building**
   - "Stale" = committed state that contradicts committed source of truth. **Uncommitted changes in the working tree are someone else's in-progress work — never modify them, never include them in your commits, never reconcile against them.** Run `git status --short` before reconciling; if any file you'd touch shows `M` or `??`, skip that file this cycle.
   - **Settle time:** do not reconcile commits younger than 2 hours. Recent commits may be mid-thought; let them settle before deciding they're stale.
   - Within those rules: fix stale status, contradictory next steps, outdated counts, broken references, missing completion notes.
   - Do not refactor broadly unless cleanup blocks forward motion.

3. **Linear/GitHub branch/worktree discipline**
   - Treat Linear as the project manager and GitHub branches/worktrees as isolation for implementation. A meaningful code feature/bug job should have one Linear issue, one matching branch, and ideally one dedicated worktree.
   - Branch naming: `r2/<linear-id>-<short-slug>` when a Linear issue exists; otherwise `r2/<yyyy-mm-dd>-<project>-<short-slug>` for approved non-Linear maintenance. Record the branch/worktree path on the Linear issue before coding when Linear is available.
   - Do not run two coding/build agents in the same working tree unless they are explicitly collaborating on the same issue and file set. Parallel agents should work in separate worktrees/branches to avoid half-baked code mixing.
   - Before starting implementation, check current branch, worktree path, `git status --short`, and upstream. If the branch does not match the active Linear issue/job, create/switch to the issue branch/worktree or stop and report branch drift.
   - Keep each branch scoped to one assignable job/chunk. When complete, verify, commit, push, update the Linear issue with branch/commit/verification, then leave merge/deploy decisions to the project contract or Justin approval gate.
   - If branch/worktree drift is detected, classify it as process drift, pause the affected build loop, and do not keep posting repeated project-topic blockers.

4. **Choose one concrete unblocked chunk**
   - Before selecting a new chunk, check today's and yesterday's memory plus the checkpoint and latest reviewer note for open reviewer-noted quality failures. Resolve the failure or explicitly carry it forward in state before starting unrelated product work. A reviewer "fix this before continuing" note supersedes the normal next-chunk suggestion.
   - If an open reviewer note names a specific file, route, CTA, broken link, or stale claim, the next builder chunk must address that exact item before normal next-chunk work. Search the full current/yesterday memory and checkpoint case-insensitively for reviewer notes (`Reviewer Finding`, `reviewer finding`, `reviewer-hard-stop`, `quality issue`) instead of relying only on bounded file heads, then grep/read the named file to prove the issue is gone. Do not treat adjacent approval-boundary polish as sufficient unless the named issue is resolved or explicitly carried forward as blocked.

   **Reviewer Finding Contract — enforceable state, not prose**
   - Reviewers are state writers, not routine broadcasters. A reviewer that creates or updates a `PARTIAL`/`FAIL` finding must persist it to the declared `open_findings_file` and return exactly `NO_REPLY`.
   - Reviewer Telegram output is allowed only for: (1) safety/external-action boundary violation, (2) the same named drift persisting across 3 consecutive reviewer evaluations without builder progress, (3) an unresolved blocker requiring Justin's decision/input, or (4) milestone/work-package retirement. Ordinary PASS/PARTIAL/FAIL, stale-source notes, missing verification, fixable builder discipline, and first/second-cycle drift stay silent and roll into the next digest.
   - A reviewer `PARTIAL` or `FAIL` creates an open finding. Canonical owner is the single file declared in the project contract as `open_findings_file: <path>`; if absent, the reviewer issuing the first `PARTIAL`/`FAIL` must declare it in the project contract in the same edit before creating findings (usually `PROJECT.md` or `STATUS.md`). If a declaration already exists, reuse it; never create a second findings file in the same project. If a `PARTIAL`/`FAIL` exists without this declaration, the next reviewer must add the declaration and convert prose findings into structured entries before builder work continues. Checkpoints, memory, chat summaries, and ad hoc run notes may point to the finding, but they are not authoritative.
   - Each open finding must include: `finding_id`, reviewer/source run, severity, exact affected files/slugs/routes/symbols, closure mode (`presence` for required fields/records that must exist, or `absence` for stale/incorrect claims that must disappear), required evidence, acceptance gate, status, and next reviewer due condition.
   - At the start of any builder run after a reviewer `PARTIAL`/`FAIL`, write a literal `Findings to resolve this cycle:` block into the declared `open_findings_file` before making changes, listing every blocking finding ID and named target. If the run lacks this enumeration there, the next reviewer must classify it as `FAIL`.
   - While blocking findings exist, the builder may only address those findings, mark a finding `blocked` with evidence, or carry it forward with a concrete next gate. Adjacent cleanup, validator-link clusters, or general quality work do not count unless they satisfy the exact finding.
   - Open reviewer findings override the 2-hour settle window for the named files/records only. Revalidate that each target still exists before editing; if a target was renamed or moved, mark the finding `needs-reviewer-revalidation` only with independent justification, an explicit new target added to the finding, and the original closure mode/acceptance gate preserved until the reviewer accepts or revises it.
   - Builders do not close their own findings. They may set `proposed-closed: <evidence ref>` after passing the required gate. The next reviewer must edit the declared `open_findings_file` to either flip the finding to `closed` with the accepted evidence, keep it `open` with the missing evidence, or mark `needs-revalidation`; reviewer prose without this state mutation is incomplete.
   - One-time rollout exception: a project may create a single `bulk-baseline` finding for inherited debt discovered on first adoption. The baseline must include `baseline_created_at`, scope, and a `no_second_baseline_without_justin_approval` line. New findings after that baseline must be individually tracked and resolved/carry-forwarded.

   - Pick the smallest useful chunk that advances the current architecture.
   - If a human approval gate is reached, first decide whether useful internal-only fallback work remains. If yes, switch to internal-only polish: docs, local UX, metadata hygiene, comparison matrices, checklists, or review packets. If no, run the Visualization / approval gate pause or Complete-block pause procedure immediately so crons do not burn empty cycles.
   - For any new directory, UI, prototype, dashboard, app surface, substantial template build, or material user-facing workflow build, the first useful chunk is the visualization/approval packet unless Justin has already approved an equivalent packet.
   - Never perform external actions, outreach, deploys, purchases, config/runtime changes, PHI/PII/EHR/payer/customer-data access, or real-money actions without explicit approval.

5. **Build with a time budget**
   - For recurring crons, target one verifiable chunk per run and stop before the next scheduled slot.
   - Avoid overlapping work. If another run is active or the prior run clearly exceeded cadence, record/skip rather than pile on.

6. **Verify before claiming progress**
   - Run the smallest meaningful gate: lint/build/test/grep/diff inspection.
   - **Touched Artifact Invariants:** every directly mutated file, route, record, slug, prompt, or state section must satisfy the project-level invariant for that surface before completion. Examples: data records have required source/date fields; routed pages pass canonical/link/page-shape/protected-preview checks; state docs have current `Last verification`/build-log entries. Transitive references are not "touched" unless directly edited.
   - If no invariant exists for a touched surface, the builder may propose the smallest invariant as an open finding for reviewer acceptance, but must not author a loose same-cycle invariant and count it as proof. Reviewer acceptance must be recorded in the declared `open_findings_file` as `invariant-accepted: <gate>` before that invariant becomes a closure gate. Until accepted, use existing validators plus explicit evidence and carry the missing invariant forward for at most two reviewer evaluations; on the third evaluation it becomes a blocking finding or must be rejected/collapsed. A reviewer evaluation means a quality reviewer run where the specific finding, invariant, or self-improvement failure mode is in scope and explicitly assessed.
   - Invariant failure is verification failure unless it is recorded as a named open finding with owner, target, and next gate. Do not bury it as vague freshness or quality debt.
   - If the cycle commits before final persistence/review, run committed-diff hygiene (`git show --check HEAD` or equivalent) before claiming the committed artifact is clean.
   - After any build/test/lint session, do a bug-check pass: read the failed or changed surface, inspect likely edge cases, and fix obvious regressions before persistence.
   - When recording route/API checks, distinguish read-only GET/fetches from local mutations such as POST/save/create/update/delete; do not call a changed surface read-only if it contains a save/create/update/delete affordance, even when the mutation is local-only and pre-existing.
   - For UI copy on mutation surfaces, do not describe controls as `metadata-only` if create/enable/run/pause/delete actions have real local side effects; say exactly what local state or saved payload action occurs, then separately name what remains approval-gated.
   - Log exact command names and pass/fail result.
   - If verification fails twice or touches 3+ files and remains unclear, stop and request fresh-eyes review.

7. **Review/edit the output**
   - Before final persistence, review every user-facing artifact, project doc, prompt, UI copy, checklist, and data change for accuracy, completeness, logic, sanity, stale claims, and approval-boundary leakage.
   - Edit once after review if the artifact is materially incomplete, internally inconsistent, vague, or overclaims progress.
   - Treat this as required even for docs-only cycles; verification proves it builds, review proves it is worth keeping.

8. **Persist state**
   - Update project status/checkpoint/memory with facts only: changed files, completed chunk, verification, blocker, next chunk.
   - A cycle is incomplete until durable state records: commit hash or explicit uncommitted reason, changed files, exact verifier commands/results, protected-preview/external-boundary checks when relevant, open findings with statuses, remaining invariant gaps, and next chunk.
   - Commit-only is not completion. State may land in the same commit or in a follow-up state commit inside the same cycle window; the cycle, not the first commit, is the unit of completion.
   - Write `cycle-complete: <commit-or-state-ref> @ <timestamp>` as the final persistence marker for every claimed cycle. If the project has `open_findings_file`, write it there; otherwise write it in the project checkpoint or status file. Missing marker means no full `PASS`; if a completion summary exists without the marker, the next reviewer must classify `PARTIAL` or `FAIL` depending on evidence.
   - Use the actual completed run timestamp for each logged cycle; do not relabel earlier cycles with the next scheduled run time or current wall-clock time.
   - After verification, replace any provisional verification placeholders in persisted logs/docs (for example "pending final verification") with the actual gate result before committing.
   - If the project source of truth has a dedicated `Last verification`, `Last verified state`, or equivalent section, prepend the latest completed cycle there; updating only the header/status line is stale-state persistence, not complete persistence.
   - If the project source of truth uses dated state/log sections such as `Current Active State — <timestamp>` and `Build Log`, update the dated header to the actual completed run time and prepend the latest Build Log entry; changing only the summary bullet while leaving the header/log stale is partial persistence.
   - If the cycle performed an approved protected preview deploy, include the deploy commit and live gate result in that same verification/state entry.
   - If a highest-precedence source-of-truth file stays stale because it is dirty or unsafe to edit, carry that forward explicitly; after two consecutive cycles, classify the next review as partial and escalate a clean-branch reconciliation need instead of treating routine product progress as fully clean.
   - Before carrying forward a stale-source-of-truth warning, re-read the named file and verify the exact stale claim is still present. If the file has already been reconciled, remove the warning from new persistence and log the stale warning as resolved instead.
   - Before persisting or announcing a root/workspace persistence-push blocker, recheck the branch against its upstream after final commits/pushes (`git rev-list --left-right --count @{u}...HEAD` or equivalent, plus `git branch -r --contains <commit>` when a commit is named). Do not carry `push deferred` / `local only` wording if the named commit is already on upstream or the branch is even.
   - Keep hot state short. Move details to project files; memory should be an index.

9. **Communicate per lane contract — silent build loops + twice-daily summaries**
   - **Project topic (default):** routine builder/reviewer loops are silent. Do not post every verified chunk, cleanup log, routine reviewer PASS/PARTIAL, or small build-session note.
   - **Interrupt Justin only for signal:** a message is allowed immediately only when there is something Justin must do, a major issue, a recurring minor issue R2 cannot fix, project/process drift, failed verification R2 cannot resolve, unsafe ambiguity, or a work package/milestone is done and its crons are paused/retired.
   - **Summaries replace build-loop chatter:** use project digest jobs for 9 AM and 9 PM ET summaries only. Each digest covers the prior 12 hours of completed work and the planned next 12 hours. If nothing material happened, return `NO_REPLY`.
   - **Completion exception:** when a finite milestone/work-package is complete and the cron set is paused/retired, tell Justin once in the project topic with the completed scope, verification, retired cron names/IDs, and successor status.
   - **Linear is the project manager, not a chat transcript:** create/update Linear issues for meaningful jobs/chunks a dev team would assign. Do not create or comment Linear records for every tiny build loop. Use issue comments for verification and state transitions; keep Telegram summaries human-level.
   - **Project Manager topic (`telegram:-1003732476287:topic:3521`):** human-gated escalations only: real blockers, explicit approval requests, user-decision questions, failed verification R2 cannot resolve, unsafe ambiguity, or architectural choices that need Justin. Never send routine build-session reports, verified-chunk summaries, or no-progress notices to the Project Manager topic.
   - **Every Project Manager message must use the PM Escalation Template** below. Project name, urgency, blocking flag, and enough context for Justin to answer without opening the project are required.
   - **Default behavior** when no escalation condition is met: return `NO_REPLY`. Routine progress belongs in the next 9 AM / 9 PM digest, not in per-loop Telegram chatter.
   - **Dual delivery rule:** when an escalation is sent to the Project Manager topic, also drop a one-line pointer in the project topic: `Blocker raised to PM: <one-line summary>`. Do not duplicate the full escalation across both topics.

10. **Self-improve the loop when evidence warrants it**
   - After reviewing recent cycles, classify each project as success / partial / failed using measurable progress: named artifact or code change, passing gate, state persisted, and good-quality output. Reviewer scoring is strict: `PASS` requires exact requested findings resolved or reviewer-accepted, state persistence complete, and touched-artifact invariants passing; `PASS-finding-scope` is allowed when named findings are fully resolved and adjacent invariant debt is explicitly carried with the two-reviewer-evaluation SLA; otherwise adjacent invariant debt caps the result at `PARTIAL`;  `PASS-with-known-debt` is allowed only for explicitly grandfathered/baseline debt; `PARTIAL` means useful work happened but findings/state/invariants remain open; `FAIL` means unrelated work, missing verification, drift, or stale-source persistence.
   - Self-improvement is evidence-backed, not vibes-backed. Tighten a project prompt after a project-specific failure; edit this skill only for cross-project failure modes or one severe concrete failure. Severe means data loss/exfiltration, external-action breach, approval-gate breach, or at least three cycles of identical named drift.
   - Every self-improvement patch must name the failure mode, point to evidence, choose the smallest procedural change, state how the next reviewer evaluations will prove it worked, include a rollback/collapse condition, and record a `self_improvement_eval_counter` entry in `open_findings_file` when present, otherwise in the same checkpoint/status file that holds `cycle-complete`. The reviewer increments/checks the counter on each evaluation where the tightening's named failure mode is in scope; at three evaluations it must mark `keep`, `revise`, or `collapse/revert`. Improvement is measured over those evaluations, not arbitrary wall-clock cycles.
   - Keep the ratchet two-way: if a tightening creates reviewer noise, blocks useful work without reducing the named failure, or does not improve the next three affected evaluations, collapse or revert it instead of adding another layer. Project-specific invariants belong in project prompts/source-of-truth; reusable enforcement architecture belongs here.
   - **Entropy budget:** if a project's docs/prompt/skill has grown by ≥30% in line count over 10 cycles without a corresponding artifact-quality improvement, the next change must be a collapse (merge files, delete redundant checks, simplify validators), not another tightening. The ratchet turns both ways.
   - Record why the loop changed in memory/project files. If the needed change is risky, ambiguous, or affects external/runtime behavior, ask Justin first.

## PM Escalation Template (required for Project Manager topic)

Every message routed to `telegram:-1003732476287:topic:3521` must use this label structure first, followed by a self-contained body. No bare prose and no log dumps.

```text
[<PROJECT>] [<URGENCY>] [<KIND>]<BLOCKING_FLAG>
<one-sentence summary>

Context:
- <what R2 was doing / what cycle produced this>
- <relevant file, route, state pointer, or exact error>
- <what R2 already tried, if anything>

Need from Justin:
- <exact question, decision, or approval scope>
- <options if helpful: A) ... B) ... C) ...>

Impact if not answered:
- <what stays blocked / what R2 will do in the meantime>
```

Field rules:
- `<PROJECT>`: short canonical project name.
- `<URGENCY>`: `URGENT` (today), `NORMAL` (this week), or `LOW` (whenever).
- `<KIND>`: `BLOCKER`, `APPROVAL`, `DECISION`, `VERIFICATION-FAIL`, `FINDING`, or `ARCH`.
- Include `Finding ID: <id>` when the escalation is tied to an open reviewer finding.
- `<BLOCKING_FLAG>`: append ` [BLOCKING]` only when the project cannot make further progress without Justin's answer; omit otherwise.
- Self-contained means Justin can answer from the message alone: include paths, exact error strings, prior decisions referenced, and the smallest useful option set.
- One escalation per message. If two unrelated blockers fire in the same cycle, send two messages.

## Cron Design Pattern

Prefer isolated persistent project sessions:

- `sessionTarget`: `session:r2-<project>-autonomous-builder`
- `payload.kind`: `agentTurn`
- `delivery.mode`: builder/digest `announce`; reviewer `none`.
- `schedule`: stagger related projects so they do not start at the same minute.
- `timeoutSeconds`: set below the interval when possible, or include explicit prompt budget.

For active build lanes, use two layers instead of one overloaded job:

1. **Builder loop** every 30–60 min, 24/7 (overnight and weekends included): cleanup-first, one chunk, verify, persist. Pause only on `.r2-pause` / `r2-pause-until:` signals or explicit maintenance windows.
2. **Quality reviewer/reconciler** every 30–60 min during intense autonomous work: review recent cycles across active projects for measurable progress and output quality, persist findings silently, then apply bounded prompt/skill tuning only when a concrete failure pattern is observed.
3. **Twice-daily digest/reconciler** at 9 AM and 9 PM ET: summarize completed work from the prior 12 hours, planned work for the next 12 hours, detect drift, identify blockers, and propose cron changes. Digest jobs are the reporting surface for normal PPO activity, and they must be written for a human project owner first: plain-language outcomes, why they matter, blockers/decisions, and what happens next. Technical evidence can be included briefly only when it helps trust the report; do not turn the digest into an internal cron/run-history trace.

Use heartbeat only for cheap checks. Do not use heartbeat for project builds.


### Claude / Opus routing guard (required for every future PPO cron)

Every future PPO cron must be safe to re-enable without creating API or OAuth spend surprises. Apply this even to disabled/staged jobs.

- Default PPO payload model: `codex` for frontier orchestration or an approved local model for simple digest/review. Never set a PPO cron payload model to `opus`, `sonnet`, `haiku`, `anthropic/*`, or any Claude/Anthropic alias.
- If the project truly needs Claude/Opus-quality help, the cron prompt must say: Claude Desktop/local subscription path only.
- The prompt must explicitly forbid Anthropic API, OpenRouter Anthropic, direct Anthropic SDK/curl, Claude Code OAuth, Claude CLI OAuth, and `sessions_spawn` with `opus` / `sonnet` / `haiku`.
- If Claude Desktop/local subscription access cannot be verified from the Mac mini desktop session, the cron must stop and report a blocker. It must not fall back to API or OAuth.
- If Claude is not needed, say so explicitly: `No Claude/Opus/Anthropic use in this cron.`
- Before enabling or re-enabling a PPO cron, audit the payload and description for the guard above.

### Delivery wiring (required for every PPO cron)

Every PPO cron must declare delivery explicitly. Implicit / fallback routing fails closed when multiple channels (telegram + discord) are configured.

- Builder cron: `delivery.mode: announce`, project topic only; prompt returns `NO_REPLY` unless immediate-signal criteria are met.
- Reviewer cron: `delivery.mode: none`; findings are persisted to the declared `open_findings_file` and summarized by digest. Use `failureAlert` for cron/runtime failures. Do not use reviewer `announce` for routine PASS/PARTIAL/FAIL.
- Digest cron: `delivery.mode: announce`, project topic only; digest never sends routine summaries to PM topic.
- `delivery.channel`: the literal channel name (e.g., `telegram`).
- `delivery.to`: the explicit chat/topic target (e.g., `telegram:-1003842389274:topic:10`), never `last`, never DM unless that IS the project's home.
- `delivery.bestEffort`: `true` for digest and for builder when appropriate so a transient delivery hiccup does not error the run.
- **PM escalation routing is a second, conditional delivery, not a replacement for `delivery.to`.** The cron's `delivery.to` always points to the project topic for routine output. Escalations are sent inside the cycle using the available messaging path with explicit target `telegram:-1003732476287:topic:3521`, formatted via the PM Escalation Template. Do not change `delivery.to` to the Project Manager topic; that would mis-route routine build reports.

Reviewer scope rule: **one reviewer per active project**, OR a reviewer prompt that explicitly skips paused projects. Combining MC + paused-TIE in one reviewer produces false-positive noise.

### Per-project reconcile cadence

The cron schedule is one cadence (e.g., 30 min); the project's declared cadence gates whether THIS run does anything. Declare in `PROJECT.md` front-matter or top section as `reconcile_cadence: <value>`:

| Value | Builder runs | Reviewer/digest runs | Use for |
|---|---|---|---|
| `active` | every cycle | every cycle | MC, TIE, anything in active build |
| `design` | only if HEAD on this folder is ≥ 2 hours old AND no `.r2-pause` | every 4th cycle | D2 in design phase, anything under heavy human review |
| `archived` | monthly | monthly | Completed projects kept for reference |
| `off` | never | never | Frozen, not in scope |

If the field is absent, default to `active` to preserve current behavior. Changing a project's cadence is a one-line edit, no cron config change.

## Digest Cron Prompt Template

Use this template for AM/PM PPO digest jobs. Do not reuse the builder template for digest crons.

```text
You are running the <PROJECT> PPO digest only. Do not build features, modify project files, create findings, reconcile state, or run implementation work.

Goal: produce a clean factual 12-hour project-topic digest only when there is material PPO signal.

Audience: Justin as a human project owner. Explain the work in plain English: what changed, why it matters, what is ready now, what is still risky or blocked, and what PPO will do next. Do not make the digest read like an internal cron/run-history trace. Use commit hashes, file paths, validator names, finding IDs, and job IDs only as short evidence after the human-level explanation, not as the main story.

Read-only inputs:
- Builder/reviewer cron run history for the prior 12 hours: <JOB_IDS>.
- Project source of truth: <FILES>.
- Checkpoint/status and today's/yesterday's memory: <FILES>.
- Declared open findings file, if present: <PATH>.
- ACTIVE_LANES/BACKLOG only if needed to detect drift.

Claude/Opus routing: No Claude/Opus/Anthropic use in this cron. Do not use Anthropic API, OpenRouter Anthropic, direct Anthropic SDK/curl, Claude Code OAuth, Claude CLI OAuth, or sessions_spawn with opus/sonnet/haiku.

Output decision tree:
1. Count material builder `cycle-complete` markers, verified commits/artifacts, closed findings, new blockers, or named project-scope decisions in the prior 12 hours.
2. If zero material completions, zero new block