# Iterate run <run-id>

- Started: <ISO-8601 UTC>
- Updated: <ISO-8601 UTC>
- repo_root: <absolute path>
- status: in-progress|done
- Target: <path> + class (ios|mac|electron|web|harness|aegis|loom|biblical|manna|generic)
- Host: cursor|grok|claude
- Mode: fix|find-only
- Lenses requested: <list>
- Standing drive auth: yes|no
- Primary workflow: <one sentence>
- Green / stop reason: <text or pending>

## Mission plan

SSOT for mission execution steps (lens order + gates). Parent TodoWrite mirrors this
table (pending→pending, in_progress→in_progress, complete→completed, blocked→cancelled,
skipped→cancelled). Child updates this table in the run log; child does not mutate
parent TodoWrite. Do not create a separate plan file.
Plan status vocabulary (distinct from finding statuses): `pending`|`in_progress`|`complete`|`blocked`|`skipped`.
Never invent plan status `capped`. At a lens's second cycle after recheck, mark the
lens step `complete` with evidence `per-lens cap reached` (remaining H/M stay `open`
unless validly deferred).
`blocked-on-Matt` is a finding status only — never use it here.
Generate rows from the **actual requested lens order** (parent plan shell on fresh;
existing rows on resume — never overwrite/reinitialize on resume).
**Presmoke terminalization:** omit Presmoke row when re-smoke is known N/A; if
present and later N/A, `skipped` with evidence `not applicable`
(Green/cap-stop/stop-early); when applicable and run, `complete` with evidence.
Then final synthesis. Before findings exist, leave Finding IDs empty; attach IDs
and concrete fix/recheck work once emitted.
Every requested lens step must be terminal (`complete`|`skipped`|`blocked`) before green.
On finalization terminalize every step by stop reason. **Psynth sequence:** template
seeds `Psynth` as `pending`; transition from `pending` or `in_progress` →
`in_progress` while constructing final synthesis → `complete` immediately before
Plan outcome; then set top-level `status: done`, Updated, and Green / stop reason.
No step stays `pending` or `in_progress`. Cap-stop names skipped lenses + cap reason
(never Green:Y). Stop-early due Matt-only blockers always yields Green:N. Implicit
resume matches exactly one `status: in-progress` for `repo_root:`; never reopen
`done`.

| Step | Lens | Objective / gate | Status | Finding IDs | Evidence / next |
|------|------|------------------|--------|-------------|-----------------|
| P1 | <requested-lens-1> | <gate for lens 1> | in_progress | | |
| P2 | <requested-lens-2> | <gate for lens 2> | pending | | |
| … | … | … | pending | | |
| Presmoke | — | final working re-smoke (omit row if not applicable) | pending | | |
| Psynth | — | final synthesis | pending | | |

## Findings backlog

Instantiate one backlog row per real finding (no phantom examples).

| id | severity | lens | status | title |
|----|----------|------|--------|-------|
| <id> | <severity> | <lens> | <status> | <title> |

## Finding detail — <id>

Exactly one detail block per backlog row (required). Missing detail = failed handoff.
Instantiate one block per real finding.

- **What:** <what>
- **Why:** <why>
- **How (planned):** <how-planned>
- **How (done):** <how-done>
- **Resolution / blocker:** <resolution-or-blocker>
- **Evidence:** <evidence> (or unverified)
- **Status:** open|logged|fixed|deferred|blocked-on-Matt|cancelled

## Round log

One block per round, same fields as the Matt-facing Round report (SKILL.md §5), including **Plan update**.
Every round including the final emits a Round report before exit finalization.
On cap-stop, name skipped lenses + cap reason in the Round report / Plan update.

### Round 1 — lens=<requested-lens-1> · mode=fix

- **Found:** 0H / 0M / 0L — ids
- **Fixed:** id — one line each (or none)
- **Deferred / blocked:** id — reason (or none)
- **Evidence:** oracle / smoke / screenshot + result (or unverified — would verify by X)
- **Open after round:** 0H / 0M
- **Plan update:** completed=… · current=… · next=… — what changed
- **Next:** round 2 lens=… | green | cap-stop | stop-early
- Drive actions announced: …

## Deferred (over budget)

- id — one-line reason parked

## Blockers for Matt

- …
