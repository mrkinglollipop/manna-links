---
name: iterate
description: >-
  Autonomous improve loop: detect target, run lens skills (working/bugs/gaps/polish),
  track findings in todos + durable run log, fix existing-surface issues, recheck.
  Use for /iterate. Not /myauditandfix (claims/correctness). Model-agnostic.
---

# Iterate

**Mode:** fix authorized by default when `/iterate` invoked for **existing surface**
plus novel run-log paths under the contract_guard allowlist. Novel product files
still need a Deliverable Contract (or other escape). Trailing `find-only` /
`audit-only` = find+log only (no writes except run log).

**vs `/myauditandfix`:** Iterate = constructive improve. myaudit = claims/lies/process. Optional post-iterate myaudit if big claims.

## 0. Parse trailing text

| Token | Meaning |
|-------|---------|
| (lens omitted) | Default lens set = **`working` only** |
| `lens=<name>` or bare `polish` / `bugs` / `gaps` / `working` | Single lens |
| `lenses=all` | `working` → `bugs` → `gaps` → `polish` |
| `lenses=a,b` | Explicit list, **left-to-right CSV order** (as written) |
| `find-only` / `audit-only` | Mode find-only |
| `resume=<path\|run-id>` | Resume run log |
| path / repo name | Force `$REPO_ROOT` / target |

## 1. Resolve host + `$REPO_ROOT`

1. Host from tools present (Task → cursor; spawn_subagent → grok; else claude). Read `.cursor/dispatch-settings.yaml` → `hosts.<host>.iterate`.
2. `$REPO_ROOT` = `git rev-parse --show-toplevel` for the **target**, not a multi-root join key.
3. Scope floor: that root only — never whole Cloud Storage.

## 2. Parent before dispatch (HARD)

Parent owns the **visible Mission plan + TodoWrite** so Matt always sees the plan
before loop work, even when child streaming is unavailable. Keep **one hire**;
never rehire per lens. **Parent owns visible TodoWrite. Child cannot mutate parent
TodoWrite.**

**Resume safety (HARD):** Resume only when explicit `resume=` resolves to a log,
or when **exactly one** matching `status: in-progress` log exists for
`repo_root:`. Match counts:
- **0** matching in-progress → `run_mode=fresh`.
- **1** matching in-progress → implicit `run_mode=resume` that log.
- **2+** matching in-progress → **stop before dispatch**; ask Matt to choose with
  explicit `resume=<path|id>` and list candidate IDs/paths. Do **not** pick
  latest mtime.
Never reopen or mutate a `status: done` run implicitly.
Do not fall back to latest-mtime completed logs.
(Finalization always sets `status: done`, so completed missions are never selected.)

**`lenses=all` / global-4 honesty (HARD):** `lenses=all` requests four lenses
(`working` → `bugs` → `gaps` → `polish`) under a **global max of 4** rounds and
**per-lens max of 2**. State this plainly in the initial Mission plan (and
command). Any lens that consumes a second cycle can force later requested lens
steps to be `skipped` at cap-stop. Round reports and the final summary / Plan
outcome must **name skipped lenses and the cap reason**. Cap-stop with skipped
lenses is **not** Green:Y. Do not raise caps.

**Branch before dispatch (or before solo Round 1):**

1. Parse lenses + mode + resume token; resolve `$REPO_ROOT`; cheap-read
   README/CLAUDE.md (or equivalent) and name/derive the primary workflow.
2. Decide `run_mode`:
   - **`run_mode=fresh`** — no explicit resume and zero matching in-progress logs:
     create a new plan shell from requested lens sequence + gates (one step per
     lens `P1`…, **omit Presmoke when re-smoke is known N/A**, else include
     Presmoke, then `Psynth`); assign stable step IDs; populate parent TodoWrite;
     print `## Mission plan` to Matt (when `lenses=all`, include the four-lenses /
     global-4 honesty line).
   - **`run_mode=resume`** — explicit valid `resume=` **or** exactly one matching
     in-progress log: read and print the **EXISTING** run-log `## Mission plan`;
     reconstruct parent TodoWrite from those rows (+ H/M findings per finding
     protocol); do **not** create a new shell or overwrite plan rows.
   - **Ambiguous (2+ in-progress)** — stop; ask Matt (see Resume safety). Do not
     dispatch.
3. Dispatch **once** (or solo) with: `$REPO_ROOT`, lens set, mode, host,
   `run_mode=fresh|resume`, run-log path/id, **exact plan rows** (stable step IDs),
   primary workflow string.

**Hire once:** Task/spawn/Agent with `subagent_type` from dispatch-settings
(`iterate` or escape + read `agents/iterate.md`). Model: omit/inherit (never pin
Opus/Fable). **Hire dead:** parent runs this skill solo (same loop; parent updates
TodoWrite live).

**Child:**
- `run_mode=fresh`: seed durable run log with received plan rows before Round 1;
  verify/adjust target class + primary workflow; record adjustments in first Round
  **Plan update**.
- `run_mode=resume`: update existing plan rows + backlog only; never overwrite or
  reinitialize the Mission plan table.
- Owns durable run-log Mission plan updates each round and returns all Round
  reports. Does **not** update parent TodoWrite.

**TodoWrite honesty / hired limitation (HARD):** In hired mode, the parent shows
the **initial** Mission plan before dispatch; per-round plan deltas live in the
run log and are **automatically replayed/rendered after return** — the child
cannot live-mutate the parent's TodoWrite UI. Do **not** promise live parent Todo updates across the hire boundary. In solo mode
the parent updates TodoWrite live each round. Matt still automatically receives:
initial plan before dispatch, full per-round plan history (replayed), and final
Plan outcome — without asking.

**Incomplete / crashed hire recovery (HARD):** If the child fails, is denied,
returns incomplete/counts-only, or omits final tables, the parent **must** inspect
the known run-log path.
- **Readable:** surface partial Mission plan, completed Round reports/findings,
  exact failure/deny reason, and `resume=<path|id>`; leave run-log
  `status: in-progress`. Reconcile parent TodoWrite **only** from that readable
  log.
- **Unreadable/missing:** report unverified failure and the expected path; do
  **not** fabricate findings or declare Green. Cancel/block the current mission
  Todo item with the failure reason (no fabricated plan/findings reconciliation).

**After child return (or solo stop):** when the run completed normally, parent
reconciles TodoWrite from final run-log plan + backlog in **one** TodoWrite
update (no plan step remains `pending`/`in_progress`); prints/reconstructs every
Round report (including Plan updates) plus `## Findings summary` +
`## Plan outcome` if child output was incomplete. Counts-only child output is
**not** an acceptable final response — use hire-recovery above. Lens skills are
**read inline** — never nested Task per lens.

## 3. Target class detect

First match (child verifies; parent may have assumed from cheap read):

| Class | Signals | Verify lane |
|-------|---------|-------------|
| iOS | xcodeproj / xcworkspace / project.yml / iOS pubspec | xc-mcp / ios-oracle |
| Mac | macOS scheme / Mac target | Mac run destination |
| Electron | electron in package.json | electron-oracle / CDP |
| Web | next/vite/react scripts | browser / Playwright |
| Aegis | Code/grok-build-harness, Aegis gui | smokes + desktop consent |
| Loom/harness | Code/loom*, .cursor-plugin hooks as target | smokes + desktop consent |
| Biblical | Code/biblical-system, Lectern | domain smoke + UI if any |
| Manna | Code/Manna, manna-links | domain smoke + UI if any |
| Generic | else | README/CLAUDE.md + primary workflow |

Mis-detect: if primary smoke fails immediately, re-read signals once; else log assumed class and continue. Ask Matt only if primary workflow cannot be named.

## 4. Run log (what / why / how)

Template: `references/run-log-template.md`.

**Path order** (`mkdir -p` first):

1. `$REPO_ROOT/.cursor/state/iterate/<run-id>.md`
2. `$REPO_ROOT/outputs/iterate/<run-id>.md`
3. Claude Mac only: `/Volumes/Cloud Storage/Claude/outputs/iterate/<run-id>.md`
4. Cloud/unknown: `/tmp/iterate/<run-id>.md` (print path in summary)

`run-id` = `YYYYMMDDTHHMMSSZ-<short>` UTC.

**Cadence:** create/seed at start (`fresh`) or open existing (`resume`); append after
each find and fix batch; Loop summary **must** print log path.

### Mission plan lifecycle (HARD)

The **Mission plan** is the mission execution plan (lens order + gates), not
speculative fix details before findings. Durable run log `## Mission plan` table
is SSOT for plan rows after seed/resume. Parent TodoWrite = visible UI mirror only.
Do **not** create a separate plan file, Plan Mode artifact, or dual SSOT.

**Two separate vocabularies (HARD — never mix):** plan-step statuses apply only to
`## Mission plan` rows; finding statuses apply only to backlog rows / detail blocks.
`blocked-on-Matt` is a **finding** status and is never a plan-step status; plan
steps use `blocked`.

**Plan table columns:** stable step ID | lens | objective/gate | status
(`pending`|`in_progress`|`complete`|`blocked`|`skipped`) | finding IDs |
evidence / next action.

**Plan status → parent TodoWrite mapping (exact):**

| Plan status | TodoWrite status |
|-------------|------------------|
| `pending` | `pending` |
| `in_progress` | `in_progress` |
| `complete` | `completed` |
| `blocked` | `cancelled` |
| `skipped` | `cancelled` |

### Finding TodoWrite protocol (HARD)

Parent Todo list = plan steps **plus** HIGH/MEDIUM finding items. Finding statuses
are `open`|`logged`|`fixed`|`deferred`|`blocked-on-Matt`|`cancelled`.

**Finding status → parent TodoWrite mapping (exact):**

| Finding status | TodoWrite status |
|----------------|------------------|
| `open` | `pending` |
| `fixed` | `completed` |
| `blocked-on-Matt` / `deferred` / `cancelled` / `logged` | `cancelled` |

Finding items are **never** `in_progress`. LOW findings may remain run-log only
unless already mirrored. The sole `in_progress` TodoWrite item is the **current
plan step**. Batch plan-step handoff and finding updates in **one** parent
TodoWrite call when solo, or **one** reconciliation call after child return when
hired. At mission end none remain `in_progress`. When advancing plan steps
(solo), update the old current item and the next item atomically in **one** TodoWrite call.

**During the loop (child → run log):** after every round / fix / recheck, update
the durable plan table (status, finding IDs, evidence/next) before emitting the
Round report. Once findings exist, attach IDs and concrete fix/recheck work.

**Every Round report** includes a concise **Plan update** (completed / current /
next step IDs + what changed).

### Round report then exit (HARD)

Every round — **including the final round** — must: (1) update the durable plan
table, (2) emit the Round report with
`Next=green|cap-stop|stop-early|round N+1`, appended to the run log.
Only **after** that final Round report may exit finalization and Mission summary
run. No exit branch may skip the final Round report.

**Plan finalization (HARD — exit path, before Mission summary):** after the final
Round report, **terminalize every plan step by stop reason** — no step may remain
`pending` or `in_progress` (including `Psynth`, which the template seeds as
`pending`):

| Stop reason | Current step | Other requested steps | `Psynth` |
|-------------|--------------|-----------------------|----------|
| **Green** | `complete` (lens/gate met); Presmoke per Presmoke terminalization | all requested applicable steps already `complete` | see Psynth sequence |
| **Cap-stop** | `complete` with cap evidence when its recheck finished, else `skipped` with cap reason; Presmoke per Presmoke terminalization | requested but unrun steps → `skipped` with cap reason (name each skipped lens) | see Psynth sequence |
| **Stop-early (Matt blocker)** | `blocked` with explicit blocker reason; Presmoke per Presmoke terminalization | remaining impossible requested steps → `blocked` or `skipped` with explicit blocker reason | see Psynth sequence |

**Presmoke terminalization (HARD):** At plan creation, **omit** the Presmoke row
when re-smoke is known N/A. If a Presmoke row exists and later proves N/A, mark
it `skipped` with explicit evidence `not applicable` on Green, cap-stop, and
stop-early. When applicable and run, mark `complete` with evidence. No auxiliary
step (Presmoke or Psynth) may remain `pending`/`in_progress` at final.

**Psynth sequence (HARD — Green, cap-stop, and stop-early):** explicitly
transition `Psynth` from `pending` **or** `in_progress` → set `Psynth` to
`in_progress` while constructing the final synthesis → set `Psynth` to
`complete` immediately before rendering **Plan outcome**. After plan-table data
is final, set run-log top-level `status: done`, refresh **Updated**, and write
**Green / stop reason**; then render Mission summary + Plan outcome. Parent's
final reconciliation maps those terminal plan states through the plan→Todo
mapping, so **no plan step remains `pending` or `in_progress`** and no
plan-mirrored parent Todo stays `pending`/`in_progress` at the final response.
(Open finding Todos may remain `pending` when Green is N.) Mark completion
durable so implicit resume can never select a completed mission.

### Finding detail completeness (HARD)

Every Findings backlog row requires exactly one `## Finding detail — <ID>` block
with fields What / Why / How (planned) / How (done) / **Resolution / blocker** /
Evidence / Status. Parent reconstructs the final Findings summary from backlog
rows + those required detail blocks. A missing detail block is a **failed child handoff**,
not permission to omit the finding from the table.

## 5. Lens loop

- Never load more than **one** lens file per find pass.
- Mission round = find→fix→recheck on current lens.
- Per-lens max **2** cycles then advance; **global max 4** (global wins).
- Cap stop: log `lenses_skipped` (name each skipped lens + cap reason); quality
  threshold for green stays zero Status=`open` **HIGH/MEDIUM** (LOW may remain
  `logged`/`open` without blocking green) — plus the requested-lens completion
  gate in §6. Cap-stop is never Green:Y.
- **Per-lens cap (HARD — never invent `capped`):** plan statuses are only
  `pending|in_progress|complete|blocked|skipped`. At a lens's **second** cycle,
  after recheck, mark that lens plan step `complete` with explicit evidence
  `per-lens cap reached` even if H/M remain. Remaining findings stay `open` (or
  become `deferred` only under existing valid policy) so they block Green and
  appear in final findings. If another requested lens is `pending` and global
  rounds remain → advance to it. Else global cap / no legal cycle → `cap-stop`.

### Round advancement (HARD — same turn, no handoff)

The mission is **one turn**. After each round's recheck, update plan → choose
`Next` → emit the Round report → act on `Next` — do not end the turn, do not ask
Matt to say "continue", never close with "next I'll run round 2".

**Per-round order (HARD):**

1. Update durable Mission plan for this round.
2. Choose `Next` **before emitting the report**, in this order:
   1. §6 Green (quality threshold **and** requested-lens completion gate) → `green`.
   2. All remaining open items are Matt-only blockers → `stop-early`.
   3. **Requested-lens advance:** current lens step is `complete` (incl. after
      second cycle with `per-lens cap reached` evidence), another requested lens
      plan step is still `pending`, and the global cap has room →
      `round N+1` on the next requested lens — **even when zero HIGH/MEDIUM are open**.
   4. Same lens still has open H/M and both its cycle cap and the global cap have
      room → `round N+1` on the same lens.
   5. Global round 4 is used, or no requested lens can legally run another cycle
      while requested/fixable work remains → `cap-stop` (name skipped lenses).
3. Emit the Round report with the resolved `Next=` value (`green` | `cap-stop` |
   `stop-early` | `round N+1`). On `cap-stop`, name skipped lenses + cap reason.
4. If `Next` is green | cap-stop | stop-early → plan finalization (Psynth
   sequence → `status: done`) → Mission summary. Else start the next round in
   this same turn.

No exit branch skips step 3. A clean round on one requested lens is **not** green
while another requested lens step is pending — `lenses=all` must run every
requested lens (or terminalize it) before green.

A turn that ends with open HIGH/MEDIUM, rounds remaining, and no Matt blocker is a
failed mission — not a checkpoint.

### Round report (mandatory, every round)

```markdown
### Round <n> — lens=<lens> · mode=<fix|find-only>
- **Found:** <h>H / <m>M / <l>L — <ids>
- **Fixed:** <id — one line each; `none` if nothing fixable>
- **Deferred / blocked:** <id — reason; `none`>
- **Evidence:** <oracle, smoke, screenshot, a11y + result; `unverified — would verify by X`>
- **Open after round:** <h>H / <m>M
- **Plan update:** completed=<ids> · current=<id> · next=<id> — <what changed>
- **Next:** round <n+1> lens=<lens> | green | cap-stop | stop-early (Matt blockers)
```

### Per find pass

1. Read `lenses/<lens>.md`.
2. Emit ≤8 findings (HIGH→MEDIUM→LOW). Overflow → Deferred (not silent drop).
3. For each finding: backlog row **and** `## Finding detail — <ID>` (incl. Resolution / blocker).
4. Fix mode: Status=`open` (parent TodoWrite maps open H/M → `pending` when solo/reconcile).
5. Find-only: Status=`logged`; no fix writes; finding todos → `cancelled` reason `find-only`.

### Fix

- Fix Status=`open` HIGH then MEDIUM; LOW only if budget remains.
- **Existing surface only** — new product surface → stop + Blockers.
- Security: never API Keys/, portfolio statements, proprietary alpha; no sensitive-screen exfil.
- Harness targets: no silent always-on hook/rule rewrites.
- Oracle: 3-run cap **per fix batch**, not whole mission.
- Log How-done + Resolution / blocker + Evidence; Status=`fixed`.

### UI drive

- App/sim/browser: announce, then drive (no ask).
- Desktop/Cursor UI: ask once unless standing auth (“good to drive until I get back”).
- Always one-line announce before drive.

### Cloud degrade

No xc-mcp / ios-oracle / Peekaboo → build-only or BLOCKED visual. Never fake VISUAL PASS. Memory/graph under `/Volumes/...` soft-fail.

### Post-polish

If polish fixes ran and `working` was in requested set (or primary path touched): cheap working re-smoke; fail → Status=`open`.

## 6. Green / stop

**Green requires ALL of:**

1. **Quality threshold** — zero Status=`open` HIGH/MEDIUM (LOW may remain), and no pending desktop-drive consent blockers.
2. **Requested-lens completion gate** — every requested lens plan step is terminal (`complete`|`skipped`|`blocked`); for normal green all requested **applicable** lenses are `complete`.
3. **No fixable requested-lens work remains** — nothing pending that a remaining round could fix within caps.

`working` gate applies only when the requested set includes `working` (omitted
default, `lens=working`, or `lenses=all`) — then verified or blocked. Find-only:
green = find+log done + zero open H/M + gate 2. Cap-stop with only
Deferred/`logged`/open-LOW is OK and uses `skipped` for unrun requested steps
(name each skipped lens + cap reason); **never Green:Y**.

This gate adds plan completion only; it never lowers the quality threshold, and
global max 4 / per-lens 2 still bound the loop. Never invent a plan status
`capped`.

**Stop-early:** remaining items all Matt-only blockers.
**stop-early due Matt-only blockers always yields `Green: N`** — never Green:Y.
Plan steps may be terminal and run-log `status: done` while mission Green remains
N because Matt blockers remain.

## 7. Mission summary (parent, once — after final Round report + plan finalization)

Emitted once after the final Round report and plan finalization (`status: done`).
Flavor-OFF for log/skill; Matt chat may use Wade over the same facts. Never in
place of the Round reports — Matt gets both the per-round trail and this closing
block.

**Automatic — no ask.** Always print the compact block, then **Findings summary**,
then **Plan outcome**. Do not wait for Matt to request findings. Parent
reconstructs Round reports + both tables from the run log if the child omitted
formatting. Reconstruct Findings summary from backlog + required Finding detail
blocks (missing detail = failed handoff, still list the finding with
`Resolution / blocker` = `missing detail — failed handoff`).

```markdown
## Iterate mission summary
- **Target:** <repo/path> · class=<detected class>
- **Rounds used:** <n> / 4 · per-lens cycles: <lens=n, …>
- **Lenses run:** <list> · **skipped:** <list or none> — <cap reason if any>
- **Green:** Y|N — <one-line reason / stop trigger; cap-stop with skipped lenses = N; stop-early Matt blockers = N>
- **Fixed:** <count> — <ids>
- **Deferred:** <count> · **Blocked on Matt:** <count> — <ids>
- **Evidence:** <oracle / smoke / visual results, or explicit unverified>
- **Run log:** <absolute path>

## Findings summary
| ID | Severity | Lens | Status | Finding | Resolution / blocker | Evidence |
|----|----------|------|--------|---------|----------------------|----------|
| <id> | <severity> | <lens> | <status> | <plain-language title> | <fix/recheck/blocker one-liner> | <oracle/smoke/path> |

## Plan outcome
| Step | Lens | Status | Finding IDs | Evidence |
|------|------|--------|-------------|----------|
| <step-id> | <lens> | <status> | <ids or —> | <gate result> |
```

**Green line rules (HARD):** stop-early due Matt-only blockers always yields `Green: N`
(never Green:Y). Cap-stop with skipped lenses = N. Plan terminal / `status: done`
does not imply Green:Y when blockers or skipped lenses remain.

**Findings summary rules (HARD):**

- Include **EVERY** emitted finding across all lenses — HIGH/MEDIUM/LOW; fixed / deferred / blocked / cancelled / logged / open. No silent omission. No ID ranges without titles.
- One concise row per finding. Columns: ID, Severity, Lens, Status, Finding (plain-language title), Resolution / blocker, Evidence.
- If no findings: one explicit row `— | — | — | — | No findings | — | —` (do not omit the section).

**Plan outcome rules (HARD):** one row per Mission plan step with final status + evidence after plan finalization.

Blockers get one line each after the block when count > 0.

## FAQ

- Product permission cards cannot be eliminated — request `required_permissions` correctly.
- `/iterate` stamps PENDING + iterate-specific PENDING; novel run-log paths only
  (contract_guard). Novel product files still need Deliverable Contract.
- Hired workers **inherit** the parent chat's `/iterate` authorization: contract_guard
  resolves a Task child's parent transcript for contract, escape phrase (must appear
  in Matt's latest user message), and slash command. An agent's **own**-transcript
  escape phrase stays sticky across all its user messages; only **inherited** escape
  is latest-message-only. A worker returning BLOCKED for
  "no build-auth" on an **existing** file is a harness bug to report with the deny
  reason — not a reason to quietly solo.
- Run logs may live in the target repo even when the hook cwd is the Cursor workspace
  root; `.cursor/state/iterate/**` under any git top-level is allowlisted.
- Hire failure is reportable: name the tool, subagent type, and verbatim deny reason in
  the mission summary. Silent solo hides harness regressions.
