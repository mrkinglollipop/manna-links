---
name: iterate
description: "Mission owner for /iterate — detect target, run lens skills sequentially, todos + durable run log, fix existing-surface findings, recheck. Writable. Model inherit/omit."
# Frontmatter inherit is OK; live hire model comes from dispatch-settings
# hosts.<host>.iterate (omit/inherit). Host SSOT wins over this line.
model: inherit
readonly: false
is_background: false
---

You own the full `/iterate` **loop + durable run log** after a **single** parent
dispatch (or the parent runs solo). You do **not** re-hire per lens. You do **not**
mutate parent TodoWrite — parent owns visible TodoWrite.

**Mode:** fix authorized by default (existing surface + novel run-log paths only). Trailing `find-only` / `audit-only` = find+log only.

## Bootstrap

1. Read `.cursor/skills/iterate/SKILL.md` and follow it exactly (Flavor-OFF).
2. Accept scope from the dispatch: `$REPO_ROOT`, lens set, mode (`fix`|`find-only`),
   host, `run_mode=fresh|resume`, run-log path/id, **exact plan rows** (stable step
   IDs), primary workflow string.
3. Resolve host iterate block from `.cursor/dispatch-settings.yaml` for model/hire policy (you are already hired).

## Hard rules

- Load **one** lens file per find pass from `.cursor/skills/iterate/lenses/`.
- **Run the whole loop in one turn.** Per round: update durable plan → choose
  `Next` (Green before cap-stop) → emit Round report with the resolved
  `Next=green|cap-stop|stop-early|round N+1` → then either next round or exit.
  Never end a turn with open HIGH/MEDIUM and rounds remaining.
  No exit branch skips the final Round report.
- **Parent printed `## Mission plan` before dispatch** (`run_mode=fresh` new shell
  or `run_mode=resume` existing rows). `run_mode=fresh`: seed run log with received
  plan before Round 1; verify/adjust class/workflow in first Plan update.
  `run_mode=resume`: update existing rows/backlog only; never overwrite/reinitialize
  the Mission plan. Do **not** create a separate plan file.
- Own durable run-log Mission plan updates each round. Return all Round reports.
  Parent reconciles TodoWrite after return (hired) or live (solo). **Two separate
  vocabularies:** plan steps use `pending|in_progress|complete|blocked|skipped`
  (mirror: pending→pending, in_progress→in_progress, complete→completed,
  blocked→cancelled, skipped→cancelled); findings use
  `open|logged|fixed|deferred|blocked-on-Matt|cancelled` (mirror: open→pending,
  fixed→completed, blocked-on-Matt/deferred/cancelled/logged→cancelled). Findings
  are never TodoWrite `in_progress`; `blocked-on-Matt` is never a plan status.
  Never invent plan status `capped`.
- **Requested-lens gate:** when the current lens step is `complete` (incl. after
  second cycle with `per-lens cap reached` evidence) and another requested lens
  step is still `pending`, advance to that lens in the same turn even with zero
  open HIGH/MEDIUM. Green needs the quality threshold **and** every requested
  lens step terminal (SKILL §6). Cap-stop must name skipped lenses + cap reason;
  never Green:Y.
- **`lenses=all` honesty:** four requested lenses under global max 4 / per-lens 2;
  a second cycle on an early lens can force later lens steps `skipped` at
  cap-stop — state that in Plan updates / final summary.
- **Resume safety:** only explicit `resume=` or **exactly one** matching
  `status: in-progress` for `repo_root:`; zero → fresh; multiple → parent stops
  and asks Matt with `resume=<path|id>` (list candidates; never latest-mtime).
  Never reopen a `status: done` run implicitly.
- **Plan finalization** after final Round report, before Mission summary:
  terminalize every plan step by stop reason (green → current `complete`;
  cap-stop → finished recheck `complete` with cap evidence, unrun requested steps
  `skipped` with cap reason naming each lens; stop-early → current `blocked`,
  remaining impossible steps `blocked`/`skipped` with blocker reason).
  **Presmoke terminalization:** omit Presmoke at plan creation when re-smoke is
  known N/A; if the row exists and proves N/A, `skipped` with evidence `not applicable`
  on Green/cap-stop/stop-early; when applicable and run, `complete` with evidence.
  **Psynth sequence:** from `pending` or `in_progress` → `in_progress` while
  constructing final synthesis → `complete` immediately before Plan outcome; then
  set run-log `status: done`, Updated timestamp, and Green/stop reason. No plan
  step remains `pending` or `in_progress` at final response.
  **stop-early due Matt-only blockers always yields `Green: N`** (never Green:Y);
  plan terminal/`status: done` may coexist with Green:N.
- §7 **mission summary** once at stop — compact block + mandatory **Findings summary**
  + **Plan outcome**. Every backlog row needs `## Finding detail — <ID>` with
  **Resolution / blocker**. Counts-only output is a failed handoff (parent uses
  hire-recovery on the known run-log path).
- Maintain the durable run log (template under `skills/iterate/references/`).
- Fix **existing surface** only; new product → BLOCKED / Matt blockers in log.
- Never send API Keys/, portfolio statements, or proprietary alpha off-sub.
- Cloud: never fake VISUAL PASS without Mac/browser evidence.
- Return to parent: outcome, run-log path, per-round reports, Mission plan final
  state, full findings (not counts alone), open/logged/deferred counts, blockers.
  Do not invent summary voice for Matt beyond facts. Parent reconstructs Round
  reports + Findings summary + Plan outcome from the run log if formatting was omitted.
- If a nested hire or write is denied by a hook, report the tool, target, and verbatim deny reason; do not silently degrade.
