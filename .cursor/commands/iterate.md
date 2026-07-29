---
name: iterate
description: Autonomous improve loop — lenses working/bugs/gaps/polish, todos + durable run log, fix existing surface (max 4 rounds)
---

# Iterate

**Authoritative contract:** `.cursor/skills/iterate/SKILL.md`. This command **authorizes the fix phase** (unless trailing `find-only` / `audit-only`).

Iterate = constructive improve. It is **not** the session-audit command (claims and
correctness). Command docs are injected into the user turn, so this file never spells
sibling slash names — that made every iterate turn read as an audit session.

## Where this command lives

1. `~/.cursor/commands/` (global — after sync `--local`)
2. `<workspace-root>/.cursor/commands/` (project / Cloud)

```bash
bash "/Volumes/Cloud Storage/Claude/.cursor/scripts/sync-harness.sh"
```

## Trailing text

| Token | Effect |
|-------|--------|
| (none) | Lens = `working` only |
| `lenses=all` | working → bugs → gaps → polish (**4 lenses**, global max **4** rounds, per-lens max **2** — a second cycle on an early lens can force later lenses skipped at cap-stop; not Green:Y) |
| `lenses=a,b` | Explicit list, left-to-right CSV order |
| `lens=<name>` / bare `polish` / `bugs` / `gaps` / `working` | Single lens |
| `find-only` / `audit-only` | No fix writes (run log still written) |
| `resume=<path\|id>` | Resume run log (required when 2+ in-progress runs match) |
| path / repo | Target `$REPO_ROOT` |

## Pipeline

1. Resolve host + `$REPO_ROOT` (git top-level of target).
2. **Parent before dispatch:** parse lenses; resolve fresh vs resume; cheap-read
   primary workflow. Implicit resume only if **exactly one** matching
   `status: in-progress` for `repo_root:`; zero → fresh; **2+ → stop and ask**
   Matt with explicit `resume=<path|id>` (list candidates; never latest-mtime).
   `run_mode=fresh` → create/print plan shell + parent TodoWrite (state
   `lenses=all` / global-4 honesty when applicable). `run_mode=resume` →
   read/print EXISTING run-log Mission plan + reconstruct TodoWrite (do not
   overwrite plan rows).
3. Dispatch **once** to `iterate` with `run_mode=fresh|resume`, run-log path/id, and
   exact plan rows (`hosts.<host>.iterate`, model omit/inherit). Hire fail → solo
   (parent TodoWrite live). Child seeds (fresh) or updates existing rows (resume);
   never mutates parent TodoWrite. Hired mode = initial plan + automatic replay
   after return — do not promise live parent Todo updates.
4. Child owns lens loop + durable plan/log updates + Round reports + fixes. Parent:
   desktop-drive consent if needed; after return, reconcile TodoWrite once from
   final run log; reconstruct every Round report + **Findings summary** +
   **Plan outcome** + **run-log path** if child output was incomplete. If child
   fails/denied/incomplete/counts-only: inspect known run-log path — readable →
   surface partial plan/rounds + resume id, leave `in-progress`; unreadable →
   unverified failure (no fabricated Green/findings).
5. Lens skills read **inline** — no nested Task per lens.
6. **One turn, many rounds.** Every round (incl. final) updates plan then emits
   Round report before any exit finalization / Mission summary (SKILL §5).
   Cap-stop names skipped lenses; never invent plan status `capped`.

**You always receive (no ask):** initial `## Mission plan` **before** dispatch/Round 1;
full per-round Round reports with Plan update (replayed by parent after hire if
needed); automatic final compact summary + Findings summary (every finding with
title/severity/status/resolution/evidence) + Plan outcome; run-log path.

## Host dispatch

| Host | Tool | Type | Model |
|------|------|------|-------|
| cursor | Task | `iterate` (escape `generalPurpose` + agent md) | omit / inherit |
| grok | spawn_subagent | general-purpose + agent md | omit always |
| claude | Agent | general-purpose + agent md | omit always |

## Green / stop

See skill. Round caps: per-lens 2, global 4 (global wins). Stop-early when only Matt
blockers remain — **stop-early due Matt-only blockers always yields `Green: N`**
(never Green:Y); plan may be terminal/`status: done` while Green remains N.
Presmoke: omit when re-smoke known N/A; if present and N/A → `skipped` (`not
applicable`); when run → `complete` with evidence.
Psynth finalization: `pending`/`in_progress` → `in_progress` while synthesizing → `complete` before Plan outcome.
