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

## 2. Dispatch or solo

- **Hire once:** Task/spawn/Agent with `subagent_type` from dispatch-settings (`iterate` or escape + read `agents/iterate.md`). Model: omit/inherit (never pin Opus/Fable).
- **Hire dead:** parent runs this skill solo (same procedure).
- Agent owns the loop; parent owns Loop summary + desktop-drive consent. Lens skills are **read inline** — never nested Task per lens.

## 3. Target class detect

First match:

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

**Resume:** `resume=` path/id; else glob log root for `status: in-progress` matching `repo_root:`; else latest mtime. Rebuild todos from Status=`open` (fix) or `logged` (find-only).

**Cadence:** create at start; append after each find and fix batch; Loop summary **must** print log path.

## 5. Lens loop

- Never load more than **one** lens file per find pass.
- Mission round = find→fix→recheck on current lens.
- Per-lens max **2** cycles then advance; **global max 4** (global wins).
- Cap stop: log `lenses_skipped`; green iff zero Status=`open` **HIGH/MEDIUM**
  (LOW may remain `logged`/`open` without blocking green). TodoWrite mirrors open H/M.

### Round advancement (HARD — same turn, no handoff)

The mission is **one turn**. After each round's recheck, evaluate the exit test and
act immediately — do not end the turn, do not ask Matt to say "continue", never
close with "next I'll run round 2".

**Exit test (in order):**

1. Green per §6 → stop, emit final summary.
2. Global round 4 used, or every requested lens exhausted its 2 cycles → cap-stop, emit final summary.
3. All remaining open items are Matt-only blockers → stop-early, emit final summary.
4. Otherwise → print the Round report, then **start the next round in this same turn**
   (next lens if the current lens hit its 2-cycle cap or produced no open H/M; same
   lens otherwise).

A turn that ends with open HIGH/MEDIUM, rounds remaining, and no Matt blocker is a
failed mission — not a checkpoint.

### Round report (mandatory, every round)

Print after every round, before starting the next. Same block appends to the run log.

```markdown
### Round <n> — lens=<lens> · mode=<fix|find-only>
- **Found:** <h>H / <m>M / <l>L — <ids>
- **Fixed:** <id — one line each; `none` if nothing fixable>
- **Deferred / blocked:** <id — reason; `none`>
- **Evidence:** <oracle, smoke, screenshot, a11y + result; `unverified — would verify by X`>
- **Open after round:** <h>H / <m>M
- **Next:** round <n+1> lens=<lens> | green | cap-stop | stop-early (Matt blockers)
```

### Per find pass

1. Read `lenses/<lens>.md`.
2. Emit ≤8 findings (HIGH→MEDIUM→LOW). Overflow → Deferred (not silent drop).
3. Fix mode: Status=`open`; TodoWrite mirrors open H/M.
4. Find-only: Status=`logged`; no fix writes; todos completed/cancelled reason `find-only`.

### Fix

- Fix Status=`open` HIGH then MEDIUM; LOW only if budget remains.
- **Existing surface only** — new product surface → stop + Blockers.
- Security: never API Keys/, portfolio statements, proprietary alpha; no sensitive-screen exfil.
- Harness targets: no silent always-on hook/rule rewrites.
- Oracle: 3-run cap **per fix batch**, not whole mission.
- Log How-done + Evidence; Status=`fixed`.

### UI drive

- App/sim/browser: announce, then drive (no ask).
- Desktop/Cursor UI: ask once unless standing auth (“good to drive until I get back”).
- Always one-line announce before drive.

### Cloud degrade

No xc-mcp / ios-oracle / Peekaboo → build-only or BLOCKED visual. Never fake VISUAL PASS. Memory/graph under `/Volumes/...` soft-fail.

### Post-polish

If polish fixes ran and `working` was in requested set (or primary path touched): cheap working re-smoke; fail → Status=`open`.

## 6. Green / stop

**Green:** zero Status=`open` HIGH/MEDIUM; no pending desktop-drive consent blockers; `working` gate only when requested set includes `working` (omitted default, `lens=working`, or `lenses=all`) — then verified or blocked. Find-only: green = find+log done + zero open H/M. Cap-stop with only Deferred/`logged`/open-LOW OK.

**Stop-early:** remaining items all Matt-only.

## 7. Mission summary (parent, once — after the loop stops)

Emitted once when the exit test fires, after the last Round report. Flavor-OFF for
log/skill; Matt chat may use Wade over the same facts. Never in place of the Round
reports — Matt gets both the per-round trail and this closing block.

```markdown
## Iterate mission summary
- **Target:** <repo/path> · class=<detected class>
- **Rounds used:** <n> / 4 · per-lens cycles: <lens=n, …>
- **Lenses run:** <list> · **skipped:** <list or none>
- **Green:** Y|N — <one-line reason / stop trigger>
- **Fixed:** <count> — <ids>
- **Deferred:** <count> · **Blocked on Matt:** <count> — <ids>
- **Evidence:** <oracle / smoke / visual results, or explicit unverified>
- **Run log:** <absolute path>
```

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
