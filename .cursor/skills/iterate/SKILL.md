---
name: iterate
description: >-
  Deep iterate: parent-owned parallel finders, serial fixer, lenses
  working/bugs/gaps/polish/product, per-lens budgets, tool utilization.
  Use for /iterate. Not session-audit (claims/correctness). Model-agnostic.
---

# Iterate

**Mode:** fix authorized by default when `/iterate` invoked for **existing surface**
plus novel run-log paths under the contract_guard allowlist. Trailing `find-only` /
`audit-only` = find+log only (no fix writes except run log).

**vs session-audit:** Iterate = constructive improve on existing surface. Session audit
= claims/lies/process correctness.

## 0. Parse trailing text

| Token | Meaning |
|-------|---------|
| (lens omitted) | Default lens set = **`working` only** |
| `lens=<name>` or bare `polish` / `bugs` / `gaps` / `working` / `product` | Single lens |
| `lenses=all` | `working` → `bugs` → `gaps` → `polish` → `product` |
| `lenses=a,b` | Explicit list, **left-to-right CSV order** |
| `find-only` / `audit-only` | Find-only mode |
| `resume=<path\|run-id>` | Resume run log |
| path / repo name | Force `$REPO_ROOT` / target |

## 1. Resolve host + `$REPO_ROOT`

1. Host from tools present (Task → cursor; spawn_subagent → grok; else claude). Read
   `.cursor/dispatch-settings.yaml` → `hosts.<host>.iterate_finder` /
   `iterate_fixer`.
2. `$REPO_ROOT` = `git rev-parse --show-toplevel` for the **target**.
3. Scope floor: that root only — never whole Cloud Storage.

## 2. Parent before dispatch (HARD)

**Parent owns the mission.** Parent seeds run log, fans out finders, merges, fixes,
rechecks, and finalizes in **one turn** (Parent fan-out / multi-wave one turn).
**Parent updates TodoWrite live between phases.** Hired finders/fixers never mutate
parent TodoWrite or the run log (**durable-write ban** on finder; fixer returns
facts only).

**Mission ceilings:**

- `schema_version: 2`
- max finder dispatches = `2 × N_requested_lenses` in fix mode (Wave A find + Wave B
  recheck per lens); **find-only** max = `N_requested_lenses` (Wave A only)
- max fixer batches = **2**
- max oracle runs = **3 per fix batch**
- Per-lens max **2** passes (find + recheck) in fix mode; find-only = **1** pass/lens

**Resume safety (HARD):** Resume only when explicit `resume=` resolves to a log, or
when **exactly one** matching `status: in-progress` log exists for `repo_root:`.
Match counts:

- **0** matching in-progress → `run_mode=fresh`.
- **1** matching in-progress → implicit `run_mode=resume` that log.
- **2+** matching in-progress → **stop before dispatch**; ask Matt to choose with
  explicit `resume=<path|id>` and list candidate IDs/paths.

Never reopen or mutate a `status: done` run implicitly.
Do not fall back to latest-mtime completed logs.

**`lenses=all` / per-lens budget honesty (HARD):** `lenses=all` requests five lenses
(`working` → `bugs` → `gaps` → `polish` → `product`) with **per-lens max 2 passes**
(find + recheck). State plainly in the initial Mission plan. A lens that exhausts its
second pass marks the plan step `complete` with evidence `per-lens budget reached` even
if H/M remain. Mission may end in `budget-stop` when finder/fix budgets exhaust —
name skipped lenses and reason; **not** Green:Y.

**Branch before dispatch:**

1. Parse lenses + mode + resume; resolve `$REPO_ROOT`; cheap-read primary workflow.
2. Decide `run_mode`:
   - **`run_mode=fresh`** — create plan shell (`schema_version: 2`), one row per
     requested lens + optional Presmoke + Psynth; parent TodoWrite; print
     `## Mission plan`.
   - **`run_mode=resume`** — read EXISTING Mission plan; reconstruct TodoWrite; do not
     overwrite plan rows.
   - **Ambiguous (2+ in-progress)** — stop; ask Matt (see Resume safety).
3. Seed durable run log from `references/run-log-template.md` before Wave A.

**Incomplete / crashed hire recovery (HARD):** If a finder/fixer fails, is denied, or
returns incomplete schema, parent inspects the known run-log path.

- **Readable:** surface partial plan/phases + `resume=<path|id>`; leave
  `status: in-progress`.
- **Unreadable:** unverified failure; do not fabricate Green or findings.

## 3. Target class detect

| Class | Signals | Verify lane |
|-------|---------|-------------|
| iOS | xcodeproj / xcworkspace / project.yml | xc-mcp / ios-oracle |
| Mac | macOS scheme / Mac target | Mac run destination |
| Electron | electron in package.json | electron-oracle / CDP |
| Web | next/vite/react scripts | browser / Playwright |
| Aegis | Code/grok-build-harness | smokes + desktop consent |
| Loom/harness | loom*, .cursor-plugin hooks | smokes + desktop consent |
| Biblical | Code/biblical-system | domain smoke + UI |
| Manna | Code/Manna | domain smoke + UI |
| Generic | else | README/CLAUDE.md + workflow |

Mis-detect: re-read signals once on immediate smoke fail; else log assumed class.

## 4. Run log

Template: `references/run-log-template.md` (`schema_version: 2`).

**Path order** (`mkdir -p` first):

1. `$REPO_ROOT/.cursor/state/iterate/<run-id>.md`
2. `$REPO_ROOT/outputs/iterate/<run-id>.md`
3. Claude Mac: `/Volumes/Cloud Storage/Claude/outputs/iterate/<run-id>.md`
4. Cloud/unknown: `/tmp/iterate/<run-id>.md`

`run-id` = `YYYYMMDDTHHMMSSZ-<short>` UTC.

Parent appends after each **phase**; mission summary prints log path.

### Mission plan lifecycle (HARD)

Mission plan = lens order + gates (not speculative fixes). Run-log `## Mission plan`
is SSOT. Parent TodoWrite mirrors plan rows only.

**Plan statuses:** `pending`|`in_progress`|`complete`|`blocked`|`skipped` — never
`capped`.

**Finding statuses:** `open`|`logged`|`fixed`|`deferred`|`deferred_overflow`|`blocked-on-Matt`|`cancelled`.
`blocked-on-Matt` is never a plan-step status.

**Two separate vocabularies (HARD — never mix):** plan-step statuses apply only to
`## Mission plan` rows; finding statuses apply only to backlog / detail blocks.
`blocked-on-Matt` is a **finding** status and is never a plan-step status.

**Plan status → parent TodoWrite mapping (exact):**

| Plan status | TodoWrite status |
|-------------|------------------|
| `pending` | `pending` |
| `in_progress` | `in_progress` |
| `complete` | `completed` |
| `blocked` | `cancelled` |
| `skipped` | `cancelled` |

### Finding TodoWrite protocol (HARD)

Parent Todo list = plan steps **plus** HIGH/MEDIUM finding items. Finding items are **never** `in_progress`. The sole `in_progress` TodoWrite item is the **current
phase** plan step. Batch plan-step handoff and finding updates atomically in **one** TodoWrite call.

| Finding status | TodoWrite status |
|----------------|------------------|
| `open` | `pending` |
| `fixed` | `completed` |
| `blocked-on-Matt` / `deferred` / `deferred_overflow` / `cancelled` / `logged` | `cancelled` |

**Plan finalization (HARD — exit path, before Mission summary):** after the final
Phase report, **terminalize every plan step by stop reason** — no plan step remains `pending` or `in_progress`. Then set run-log top-level `status: done`.

Never invent plan status `capped`. Use `per-lens budget reached` evidence on
`complete` when a lens hits its second pass with open H/M.

## 5. Lens loop

Alias for the parent multi-wave pipeline (finders are no longer inline).

## 5b. Parent multi-wave pipeline (one turn)

```text
Wave A: parallel iterate-finder (one dispatch per requested lens, pass=find)
  → parent merge + dedupe → Phase report A
Fix batch 1: iterate-fixer (merged backlog) → Phase report B
Wave B: parallel iterate-finder (pass=recheck, touched lenses only)
  → merge → Phase report C
Fix batch 2 (optional, budget remaining): iterate-fixer → Phase report D
Finalize: Presmoke (if row exists) → Psynth → Mission summary
```

**Find-only mode (HARD):** skip fixer and Wave B recheck/fix; findings Status=`logged`
(not `open`); no fix writes. Run log still written. Finder budget = `N_requested_lenses`
only. **Green in find-only:** zero Status=`open` H/M is automatic when all findings are
`logged`; still require every requested lens plan step terminal — Mission summary must
state `find-only` (audit surface) so Green:Y is not mistaken for a clean fix mission.

### Finder structured return schema (HARD)

Each finder returns the block defined in `.cursor/agents/iterate-finder.md` (plugin
SSOT: `.cursor-plugin/agents/iterate-finder.md`). Parent validates coverage inventory
fields, **Tools used**, **Tools missing**, findings table, recheck table (Wave B).

**Task enum escape (HARD):** if host Task rejects `subagent_type: iterate-finder` /
`iterate-fixer`, dispatch `escape_type` from `dispatch-settings.yaml`
(`generalPurpose` on cursor) with the agent md as ROLE brief — do not abort Wave A.

### Fixer structured return schema (HARD)

Each fixer returns the block defined in `.cursor/agents/iterate-fixer.md`. Parent
copies paths/IDs/oracle runs into the Phase report.

### Merge dedupe rules

See `references/finder-common.md`:

1. Same path + same defect → one finding; highest severity wins.
2. Same title, different paths → separate IDs.
3. Cross-lens overlap → keep both when lenses differ; link in detail.
4. Parent assigns durable merged IDs `<lens>-<sequence>` at merge (e.g. `working-1`).

**Mission merged open H/M cap (HARD):** parent keeps at most **12** merged Status=`open`
HIGH/MEDIUM findings in the active backlog. Overflow → status `deferred_overflow` and
list under run-log `## Deferred (over budget)` (see `finder-common.md`). Cap does not
apply to product `logged` proposals.

**Below-floor coverage escalation (HARD):** if any finder returns Coverage vs floor =
`below-floor` (or `partial` on a required floor), parent must either (a) ensure a
MEDIUM+ `below-coverage-floor` finding is in the merged backlog, or (b) refuse Green
and set `Green: N` with that coverage miss named — never silently Green past a
below-floor return.

### Fixer batches (HARD)

Parent hires `iterate-fixer` serially (max **2** batches). Each batch receives merged
`open` HIGH/MEDIUM backlog (LOW if budget). Fixer returns paths edited, IDs fixed,
oracle runs, remaining open — parent logs to Phase report. **product** findings are
never sent to fixer (**product lens never blocks Green**).

### Phase report (mandatory, every phase)

```markdown
### Phase <label> — <wave|fix> · lenses=<list>
- **Found / Fixed / Recheck:** <counts + ids>
- **Coverage:** <floor summary>
- **Tools used:** <list>
- **Tools missing:** <list>
- **Evidence:** <oracle / smoke / a11y>
- **Open after phase:** <h>H / <m>M
- **Plan update:** completed=<ids> · current=<id> · next=<id>
- **Next:** <next phase> | green | budget-stop | stop-early
```

### Tool utilization contract

Every finder pass must attempt class-appropriate tools (`references/finder-common.md`
tool matrix) before file-only findings. Unavailable tools → **Tools missing** +
explicit degrade — never fake VISUAL PASS.

### Per-lens budget

Each requested lens gets at most **2** finder passes in fix mode (find + recheck). After
recheck on pass 2, mark lens plan step `complete` with `per-lens budget reached` if
H/M remain (`open` in fix mode; stay `logged` in find-only). Advance to next lens or
`budget-stop` when finder budget is exhausted (`2 × N` fix mode; `N` find-only).

### Presmoke terminalization (HARD)

At plan creation, **omit** the Presmoke row when re-smoke is known N/A. If Presmoke
row exists and later proves N/A, mark `skipped` with evidence `not applicable`. When
applicable and run, mark `complete` with evidence.

### Psynth sequence (HARD — Green, budget-stop, and stop-early)

Explicitly transition `Psynth` from `pending` **or** `in_progress` → set `in_progress`
while constructing final synthesis → `complete` immediately before **Plan outcome**.
Then set run-log `status: done`, Updated, Green / stop reason.

### Finding detail completeness (HARD)

Every backlog row needs `## Finding detail — <ID>` with What / Why / How (planned) /
How (done) / **Resolution / blocker** / Evidence / Status.

## 6. Green / stop

**Green requires ALL of:**

1. Zero Status=`open` HIGH/MEDIUM (product `logged` items excluded —
   **product lens never blocks Green**). Find-only: findings are `logged`, so #1 is
   satisfied by mode; still label Green reason `find-only` (not a fix clean).
2. Every requested lens plan step terminal (`complete`|`skipped`|`blocked`).
3. No fixable work remains within budgets (find-only: no fixer hired ⇒ #3 holds).

**budget-stop:** finder or fixer ceiling hit; name skipped lenses; Green:N.

**Stop-early:** remaining items are Matt-only blockers.
**stop-early due Matt-only blockers always yields `Green: N`** — never Green:Y.

**Green line rules (HARD):** stop-early due Matt-only blockers always yields `Green: N`.
budget-stop with skipped lenses = N. Plan `status: done` does not imply Green:Y.

## 7. Mission summary (parent, once)

After final Phase report + plan finalization:

```markdown
## Iterate mission summary
- **Target:** <repo> · class=<class>
- **Phases:** <list> · **finder dispatches:** <n> / <2×N> · **fixer batches:** <n> / 2
- **Lenses run:** <list> · **skipped:** <list or none>
- **Green:** Y|N — <reason>
- **Fixed:** <count> — <ids>
- **Product proposals (logged):** <count> — <ids>
- **Evidence:** <oracles / smokes / unverified>
- **Run log:** <absolute path>

## Findings summary
| ID | Severity | Lens | Status | Finding | Resolution / blocker | Evidence |

## Plan outcome
| Step | Lens | Status | Finding IDs | Evidence |
```

Include **every** finding. Product proposals appear with Status=`logged`.

## FAQ

- Parent hires `iterate-finder` (readonly) and `iterate-fixer` (writable) per
  `dispatch-settings.yaml`; legacy `iterate` agent is thin redirect to fixer if
  escape-hired — parent still owns loop.
- `iterate-finder`: **durable-write ban** — no run log, no TodoWrite, no nested Task.
- Contract_guard: `/iterate` authorizes existing-surface fixes + novel run-log paths.
- Hire failure: report tool, type, deny reason; parent may solo finder/fixer roles.
