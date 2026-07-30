# Iterate finder common reference

Shared by all lenses and `iterate-finder` dispatches. Parent merges parallel finder
returns; finders never write the run log (**durable-write ban**).

## Inventory (every find pass)

Before emitting findings, record coverage in the finder return:

| Field | Content |
|-------|---------|
| Surfaces enumerated | Files, routes, schemes, scripts, hooks, UI entry points |
| Surfaces exercised | Built, launched, driven, or read with evidence |
| Surfaces skipped | Name + reason (Cloud degrade, permission, N/A) |
| Coverage vs floor | met / partial / below-floor |
| journey_step rows | For `working`: 3–7 ordered steps with `journey_step: true` |

Below-floor coverage on a required floor → emit at least one MEDIUM
`below-coverage-floor` finding (or HIGH if primary workflow untouched).

## Severity

| Level | When |
|-------|------|
| HIGH | Broken primary workflow, crash, wrong result, security floor risk, false Green risk |
| MEDIUM | Correctness edge, missing stated requirement, partial coverage floor miss |
| LOW | Polish, consistency, minor UX, non-blocking proposals |

Cap **8** findings per finder dispatch. Parent mission finding cap **12** merged
open H/M (overflow → `deferred_overflow` / `## Deferred (over budget)`).

## Evidence

Every finding needs verifiable evidence:

- Oracle command + exit + tail
- File cite `path:line`
- a11y tree / screenshot reference (never fake VISUAL PASS on Cloud)
- `unverified — would verify by X` when blocked

## Tool matrix (all classes)

Route tools by `target_class` (iterate SKILL §3). Use the highest row that applies;
record **Tools used** and **Tools missing** in every finder return.

| Class | Primary tools | Visual fallback | Degrade |
|-------|---------------|-----------------|---------|
| iOS | xc-mcp a11y-first (`idb-ui-describe`, `workflow-build-and-run`) | xc-mcp screenshot | build-only; BLOCKED visual |
| Mac | xc-mcp Mac destination | screenshot | build-only |
| Electron | electron-oracle / CDP | browser MCP screenshot | build + CDP if available |
| Web | browser MCP / Playwright MCP | screenshot | build/lint only |
| Aegis / Loom / harness | smokes, `scripts/harness_doctor.py`, hook inventory | desktop consent tools | static read + smokes |
| Biblical / Manna | domain smokes, app launch if present | UI drive when authorized | read + smoke |
| Generic | README/CLAUDE.md, pytest/build per repo | browser if web-ish | read + cheapest oracle |

**Tool utilization contract (HARD):** each finder pass must attempt class-appropriate
tools before file-only findings. If a tool class is unavailable, list it under
**Tools missing** and degrade explicitly — never imply visual pass without evidence.
Tell Matt when a required tool is missing and name the best available alternative.

## Coverage floors (by lens)

Floors are minimum surfaces that must be enumerated and (when applicable) exercised.

### working

Parent/finder decomposes the primary workflow into an ordered journey checklist of
**3–7 steps** (entry → success criteria → exit; include one empty/error/persistence
step when applicable).

| Floor | Requirement |
|-------|-------------|
| Journey checklist | 3–7 steps listed in coverage.inventory with `journey_step: true` |
| Exercise floor | ≥80% of those steps driven **or** each undriven step has BLOCKED + tool-missing reason |
| Launch/build | Class verify lane executed or BLOCKED with reason |
| Evidence | Pass/fail recorded — no diff-only "working" |

### bugs

| Floor | Requirement |
|-------|-------------|
| Category coverage | ≥1 checked category per inventoried surface: correctness \| edge \| silent-failure \| a11y-functional |
| Hot paths | Primary workflow + files touched in recent session context |
| Oracles | At least one automated check (test/build/lint) when repo provides |

### gaps

From named intent sources, list every discrete requirement.

| Floor | Requirement |
|-------|-------------|
| Intent sources | README, CLAUDE.md, plan, thread contract — cite which |
| P0 | Primary-workflow / must-ship for the stated product to work |
| P1 | Explicitly listed secondary requirements in the same intent doc/thread |
| Check floor | 100% of P0+P1 checked or N/A with cite |
| Unlabeled intent | Treat all primary-workflow requirements as P0 and all other cited bullets as P1 |
| Unnamed intent | Log blocker finding — do not invent scope |

### polish

| Floor | Requirement |
|-------|-------------|
| Design checklist | Items marked pass/fail/N/A per applicable surface |
| Cross-surface | ≥1 consistency check across surfaces |
| Pattern match | Compare to existing in-repo patterns |
| Skill routing | Route into `ui-design-quality` / `swiftui-pro` / `electron-oracle` / `art-vision` when class matches |

### product

| Floor | Requirement |
|-------|-------------|
| Friction map | Primary-journey friction map + scored proposals **or** explicit "none found" with evidence |
| Non-blocking | Proposals only — never auto-fix |
| Green | **product lens never blocks Green** — findings use `logged` status |

## Merge dedupe rules (parent)

When merging parallel finder returns:

1. **Same path + same defect** → one finding; highest severity wins; merge evidence.
2. **Same title, different paths** → keep separate IDs.
3. **Cross-lens duplicate** → keep both if lenses differ; parent notes link in detail.
4. **temp_id** → parent assigns durable merged IDs `<lens>-<sequence>` at merge
   (e.g. `working-1`). Do not invent a parallel `ITER-<n>` scheme.
5. Dedupe by symptom / root cause / surface before fixing.
6. Recheck pass updates existing IDs only; new regressions get new IDs.

## Find-only mode

Finders behave identically; parent sets finding status `logged` (not `open`). Fixer is
not hired.
