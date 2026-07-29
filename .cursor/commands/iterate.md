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
| `lenses=all` | working → bugs → gaps → polish |
| `lenses=a,b` | Explicit list, left-to-right CSV order |
| `lens=<name>` / bare `polish` / `bugs` / `gaps` / `working` | Single lens |
| `find-only` / `audit-only` | No fix writes (run log still written) |
| `resume=<path\|id>` | Resume run log |
| path / repo | Target `$REPO_ROOT` |

## Pipeline

1. Resolve host + `$REPO_ROOT` (git top-level of target).
2. Dispatch **once** to `iterate` agent per `.cursor/dispatch-settings.yaml` `hosts.<host>.iterate` (model omit/inherit). If hire fails → run skill solo.
3. Agent owns lens loop, todos, run log, fixes. Parent: desktop-drive consent if needed; final mission summary + **run-log path**.
4. Lens skills read **inline** — no nested Task per lens.
5. **One turn, many rounds.** Round report after every round; next round starts in the
   same turn until green, cap exhaustion, or Matt-only blockers (SKILL §5 exit test).

## Host dispatch

| Host | Tool | Type | Model |
|------|------|------|-------|
| cursor | Task | `iterate` (escape `generalPurpose` + agent md) | omit / inherit |
| grok | spawn_subagent | general-purpose + agent md | omit always |
| claude | Agent | general-purpose + agent md | omit always |

## Green / stop

See skill. Round caps: per-lens 2, global 4 (global wins). Stop-early when only Matt blockers remain.
