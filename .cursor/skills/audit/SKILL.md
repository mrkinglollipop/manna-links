---
name: audit
description: >-
  Read-only system, codebase, or process audit with mandatory Action summary,
  Verification ledger, and Plan completion sections on every audit-intent trigger.
  Use when Matt says audit, /audit, audit your work, session audit, assess,
  health-check, review the system, coverage audit, or asks for findings without
  fixes. Does not authorize file changes unless fix phase is authorized (/myauditandfix,
  /verify-plan, audit and fix, fix any issues). /myauditandfix = session audit + loop until green (max 4 rounds) per
  .cursor/commands/myauditandfix.md. /verify-plan = plan audit + loop plan-doc fixes until green (max 4 rounds) per
  .cursor/commands/verify-plan.md.
---

# Audit

**Mode:** read-only by default. Audit ≠ build (`master.mdc` build gate). Deliver the assessment; do not implement fixes unless fix phase is authorized: **`/myauditandfix`**, `and fix`, `fix any issues`, `fix the highs`, or a follow-up message with fix intent.

## 0. Confirm scope (one pass, then proceed)

State in one short block:
- **Target** — what is being audited (path, subsystem, diff, workflow)
- **In scope** — what you will examine
- **Out of scope** — adjacent systems you will not pull in (keep system boundaries strict — e.g. Frozen Tell ≠ Frost Line)
- **Depth** — quick scan vs thorough (default: **thorough** when Matt says "audit"; **quick** when he says "sanity check" / "spot check")

If scope is ambiguous, pick the narrowest reasonable interpretation, state it, proceed.

### Session audit + fix (`/myauditandfix`)

**Slash command:** `.cursor/commands/myauditandfix.md`. **Always** session-audit scope (invoking thread only) + **fix phase authorized** in the same turn.

Equivalent to: audit your work in this thread → mandatory report → surgical fixes to findings only. Trailing text narrows or adds target; does not skip fix unless explicitly audit-only.

### Plan verify loop (`/verify-plan`)

**Slash command:** `.cursor/commands/verify-plan.md`. Load **`.cursor/skills/verify-plan/SKILL.md`**. Plan artifact(s) in invoking thread + **plan-document fix phase authorized** (not app code). **Pipeline:** Freshness pass → dual `session-auditor` (`TRACK=plan`) → confirm-only `audit-verifier` → mandatory §4 report → fix verifier when not green. Loop audit→plan fix until green (max 4 rounds; confirm-only delta when prior confirm had zero HIGH and fix introduces no new HIGH). Equivalent to repeatedly asking "audit and verify your plan."

### Session audit (`audit your work`, `session audit`, `audit what you just did`)

**Default target:** deliverables from **the invoking thread only** — not repo-wide uncommitted diff.

| In scope | Out of scope |
|----------|--------------|
| Paths touched in this thread (`Write` / `StrReplace` / `Delete`; orchestrator + subagents) | Pre-session uncommitted WIP (paths not touched in this thread) |
| Session shell side effects (e.g. `capture_v2.py` outputs) | Unrelated workspace areas |
| Load-bearing chat claims **from this thread** (Fable rule 2) | Whole-system explore (unless session touched many areas) |
| Process rules followed in this thread (dispatch, build gate, read-only) | Drive-by fixes beyond audit findings |
| Plan / todo / approved-build scope completion (§4 Plan completion) | Items outside this thread's stated scope |

**Method:** adversarial dual-critic → verifier pipeline with **`TRACK=session`** on every dispatch (see **Session track pipeline** below). Derive session file set from **this thread's tool history**, not `git diff` alone. **Bugbot:** default **off**; opt in only if Matt asks or (rare) security-sensitive session without explicit forbid. **Skip §1 graph orient** when the target is only this thread's local files.


### Session track pipeline (dual-critic → verifier)

Professional practice: independent parallel critics with different lenses, then a separate moderator/verifier before any write. Same-context self-review is the failure mode.

**Artifact pack** (orchestrator → critics + verifier every round):
- Scope block (target, in/out, depth)
- **`TRACK=session`**
- Session file set (paths from this thread's Write/StrReplace/Delete + named shell side effects)
- Load-bearing chat claims list (verbatim or quoted)
- Plan/todo ids in thread (or "none")
- Prior round finding deltas (rounds 2–4)
- When `DELTA_CHECK=true`: paths touched by last Phase 2 fix, prior confirmed findings + clearance claims (critic reports optional)
- Oracle log tails already collected this round (if any)
- **Freshness oracle notes** (orchestrator-collected when Freshness pass ran; use "none" otherwise)
- Verifier step 1: both critic reports (bug_hunt + claim_bust) — **unless** `DELTA_CHECK=true`
- Verifier step 2 only: confirmed finding list from step 1

**Freshness pass (orchestrator — before critics when applicable):** run when session claims cite paths, versions, APIs, or external "current" facts.
1. **Always (free):** existence/help oracles for cited session paths/commands.
2. **When claims need prior context (free):** graph `search` + memory topics grep.
3. **When applicable (paid, ≤5/round, orchestrator budget):** versioned external surfaces → `.cursor/skills/web-search/SKILL.md` / `scripts/_clients`. Cite URL + date in ledger.

Paid web budget is separate from each critic's 3 oracle-run cap.

**Per round (read-only phase):**
1. Orchestrator states scope block; runs Freshness pass when applicable; attaches Freshness oracle notes. (On `DELTA_CHECK=true` rounds: may reuse last freshness notes unless session paths/claims changed.)
2. **Full confirm rounds:** **Parallel dual critics:** two `session-auditor` dispatches — `ROLE=bug_hunt` and `ROLE=claim_bust`, both with **`TRACK=session`**, **`model: "composer-2.5-fast"`**, `readonly: true`.
   - `claim_bust` includes shared freshness items: stale/false paths, outdated API/version claims, chat "verified" without evidence, scope creep vs thread intent, freshness failures.
3. **Confirm-only verifier:** `Task(subagent_type: "audit-verifier", readonly: true)` with `fix_authorized=false`, **`TRACK=session`**, and Freshness oracle notes — confirm/reject/dedupe; emit §4-ready payload. **Adjudicator model:** pin `cursor-grok-4.5-high` when Task accepts it; else `k3` when accepted; else **omit** `model` (frontmatter Grok-high / Auto+K3 inherit). Never Composer. If neither Grok nor K3 lands → STOP. Dual-mode agent cannot use frontmatter `readonly`; dispatch-level `readonly: true` is the mechanical backstop for confirm-only.
   - **Full confirm:** pass both critic reports.
   - **`DELTA_CHECK=true` (rounds 2–4 post-fix when prior confirm had zero HIGH and no new HIGH from fix):** skip dual critics; pass fix-touched paths + prior confirmed findings + clearance claims instead (see `/myauditandfix` Phase 3 / §3b).
4. **Optional bugbot:** only when Matt opted in — fold into Findings; does not replace §4.1–§4.3.
5. Orchestrator surfaces mandatory report (Action summary → Verification ledger → Plan completion → Findings) to Matt.

**Plain session audit** (`audit your work`, session audit): stop after step 5 — no fix writes.

**`/myauditandfix`:** after step 5, if not green, dispatch `audit-verifier` with `fix_authorized=true`, **`TRACK=session`**, adjudicator model (**high → k3 → omit**; never Composer), and confirmed finding list only (see §3b). **Green** requires freshness-verified ledger gate — every load-bearing claim **verified** or **blocked on Matt**; silent **unverified** fails green.

**Two-step rule (HARD):** §4 report surfaced to Matt **before any fix writes**. Verifier never confirm+write in a single Task.

**Same-turn continue (HARD):** After the §4 report is surfaced in the orchestrator message, if the audit is **not green** and **not stop-early**, **immediately** dispatch Phase 2 (`fix_authorized=true`) in the **same turn** — do **not** wait for Matt acknowledgment, “continue”, or “go fix”. `/myauditandfix` / `/verify-plan` already authorize the fix phase. Two-step only constrains **order** (report text before fix writes) and forbids confirm+fix in one Task — it is **not** a human checkpoint between phases.

**Model pins (HARD — native and escape hatch):** critics → `model: "composer-2.5-fast"`, `readonly: true` (omit denied). Adjudicator (confirm + fix) → **only** `cursor-grok-4.5-high`, `k3`, or **omit** `model` (dispatch order: high pin → k3 pin → omit). Never Composer adjudicator. Never other Grok efforts. Hook enforces this for native `audit-verifier` and verifier-shaped escape hatch. If neither Grok nor K3 can run → STOP (no Composer soft-fallback; do not kill-switch for enum gaps). Do **not** imply a mechanical pin without the hook (fail-open / kill switch). Never bare `grok-4.5-high` or Grok `*-fast`.

**Stop early — Blocked on Matt:** when every remaining HIGH/MEDIUM is Blocked on Matt and none are fixable in-session, exit the `/myauditandfix` / `/verify-plan` loop immediately (`Green: N`); do not burn further rounds. See command Phase 3.

**Escape hatch:** if Task enum lacks `session-auditor` or `audit-verifier`, use `generalPurpose` / `explore` / missing-type + read `.cursor/agents/<name>.md` with the **same** pins on the Task call. Native `subagent_type` is preferred when available (verified working in Cursor Task enum as of 2026-07-13).

Orchestrator remains hub — does not solo-audit instead of dual critics.

## 1. Orient (graph-first)

Skip this section for **session audit** when the target is only this thread's local files.

Before broad reads:
1. `PYTHONPATH=".python_libs" python3 "/Volumes/Cloud Storage/Graph/query_graph.py" search "<target>"`
2. Grep `/Volumes/Cloud Storage/Memory/conversations/topics/` for prior audits on the same target
3. Check for existing audit artifacts (`*audit*.md`, `AUDIT.md`, topic pages)

Carry forward prior findings; note what changed since last audit.

## 2. Choose audit track

| Track | When | Method |
|-------|------|--------|
| **System** | Subsystem, hooks, memory, orchestration, multi-area | 2–4 parallel `Task(subagent_type: "explore", readonly: true)` — one scope each |
| **Code** | Branch, PR, uncommitted diff, specific module | `bugbot` (+ `security-review` if security-sensitive or Matt asks) per review skills |
| **Hybrid** | Large system with code surface | Parallel explore for architecture + bugbot on diff if one exists |
| **Session** | `/myauditandfix`, `audit your work`, `session audit`, audit this thread | Dual `session-auditor` (`TRACK=session`, bug_hunt + claim_bust) → confirm-only `audit-verifier` → §4 report; `/myauditandfix` adds fix `audit-verifier` after report; bugbot opt-in only |

When ≥2 independent areas: prefer parallel Explore/Task via product Auto. Orchestrator synthesizes; subagents do not write files.

## 3. Investigate

### Evidence states (Fable rule 2 — every row, no exceptions)

| State | Meaning |
|-------|---------|
| **verified** | Checked this session; cite command, file read, or log tail |
| **unverified** | Stated in chat or docs but not checked this session |
| **inferred** | Reasonable from structure/docs; no oracle run |

Forbidden vocabulary: "should work", "looks good", "probably fine" without a state label.

### Source of truth (session build audits)

- **First user build request** in the thread (not orchestrator restatement, not subagent prompt)
- **Deliverable Contract** if one was required

### Verification discipline

- **Verification ledger** — one row per auditable claim: chat assertions, build/merge/test claims, rule-compliance claims, and positive "sound" items. Not limited to bugs.
- **Plan completion** — when the thread has a plan file, todo list, or `approved — build` scope: one row per item; surface **not started** and **partial** in Action summary.
- **Findings** — same three states in the Evidence column.
- **Documented ≠ enforced** — explicitly hunt for prose-only guarantees (rules, prompts, docs without hook/code backstop). Session pipeline + harness writes are backstopped by `audit_marker.py` pipeline gate (GREEN / Push OK requires dual-critic→confirm **Task `tool_use`** evidence in transcript when `/myauditandfix` active — assistant prose alone does not count) and `contract_shell_write.py` (Shell writes into `.cursor-plugin/` / `.cursor/{agents,skills,rules,commands,hooks,scripts}/` require contract or escape).
- **No fix loops on audit-only** — audit-only turn ends at the report unless fix phase is authorized (`/myauditandfix`, `/verify-plan`, `fix` / `and fix` / `fix any issues` / `approved — build`). **`/myauditandfix`** runs the session fix loop per `.cursor/commands/myauditandfix.md` Phase 3 (up to 4 rounds until green; confirm-only delta when prior confirm had zero HIGH and no new HIGH from fix). **`/verify-plan`** runs the plan fix loop per `.cursor/commands/verify-plan.md` Phase 3 (up to 4 rounds until green; same post-fix mode).

## 3b. Fix phase (when authorized)

When Matt runs **`/myauditandfix`** or requests audit **and** fix in the same message:
1. Complete Phase 1 of the session track pipeline first — mandatory §4 report surfaced to Matt before any writes.
2. Dispatch **`audit-verifier`** with `fix_authorized=true`, **`TRACK=session`**, adjudicator model (**pin `cursor-grok-4.5-high` when Task accepts it; else `k3`; else omit** — never Composer), and the **confirmed finding list** from confirm-only step — fix **only** those findings; no scope creep, no drive-by refactors, no orchestrator solo-fixes.
3. Re-verify fixed items via oracle/ledger updates only — **this is not a re-audit**. Update Verification ledger and Plan completion rows; note **verified** / **unverified** on each fix. Fix-agent `NEW_HIGH_FROM_FIX` / clearance notes select post-fix mode only — they do **not** mean green.
4. Small reversible edits do not require `approved — build`; new features or multi-file builds still do.
5. Orchestrator reviews verifier diff + runs oracle gate. No LOC force-dispatch.
6. **`/myauditandfix` only:** loop per `.cursor/commands/myauditandfix.md` **Phase 3** until **green** (zero HIGH/MEDIUM on the **latest confirm**, not fix self-report), **stop-early** (only Blocked-on-Matt HIGH/MEDIUM left), or **4 rounds** max, sequential. End with **Loop summary**.

**Post-fix re-audit mandatory (HARD):** after **every** Phase 2 that ran, dispatch the next round's audit (full or delta) **before** green / “no more HIGH” / Loop summary — **in the same turn**; do **not** end the turn on fix alone or wait for Matt before re-audit. `NEW_HIGH_FROM_FIX` and whether the prior confirm had any HIGH choose shape only — never skip re-audit / never equals green.

**Post-fix re-audit mode:** full re-audit when the prior confirm had any HIGH or the fix introduced any new HIGH (or HIGH regression); confirm-only `DELTA_CHECK` only when prior confirm had zero HIGH and `NEW_HIGH_FROM_FIX=false`; if delta surfaces a new HIGH, escalate to full re-audit in the same round. See `/myauditandfix` Phase 3 / `/verify-plan` Phase 3 for dispatch details.

## 4. Report format (Flavor-OFF prose)

**Mandatory on every audit-intent trigger** — session, system, code, hybrid. No compact substitute. Bugbot/security subagent output is folded into Findings; §4.1–§4.3 still required.

### 4.1 Action summary (always first)

≤6 lines. Matt should be able to stop reading here.

```markdown
# Audit: <target>
**Date:** <ISO date> · **Scope:** <one line>

## Action summary
**Verdict:** <one sentence>
**Do now:** 1. … 2. … (or "Nothing blocking — ship")
**Blocked on you:** … (or "None")
**Plan:** complete | N/A | **incomplete** — <N> open (<list ids or one-line summary>)
```

`Do now` = ordered, actionable, scoped to this audit. No drive-by refactors.

### 4.2 Verification ledger (always second)

Every load-bearing claim from chat, docs, and investigation — not only defects.

```markdown
## Verification ledger

| Claim | State | Evidence |
|-------|-------|----------|
| <what was asserted or checked> | verified / unverified / inferred | <command, path, log tail, or "not checked"> |
| Intent alignment — build matches user request primary workflow | verified / **FAILED** | Quote user noun + contract primary workflow vs what was built |
```

Include positive claims ("build passed", "PR merged", "hooks enforce X"). If a TL;DR-style bullet matters, it must appear here with a state.

**Intent alignment severity:** missing contract before first novel write = **HIGH**; contract present but primary workflow absent = **HIGH**; built feature checklist without primary workflow = **HIGH**.

### 4.2b Rule compliance (always-on sample)

After Verification ledger, for `/myauditandfix` and session audits:

```markdown
## Rule compliance (always-on sample)

| Rule file | Relevant this thread? | State | Evidence |
|-----------|----------------------|-------|----------|
| conduct | yes/no | complied / violated / N/A | contract present? done-claims verified? |
| master | yes/no | complied / violated / N/A | build gate? boundary? |
| orchestration | yes/no | complied / violated / N/A | oracle cap? delegation spec? |
| memory | yes/no | complied / violated / N/A | capture cadence? |
| personality | yes/no | complied / violated / N/A | Flavor-OFF surfaces? |
| context-compaction | yes/no | complied / violated / N/A | compact point? |
| workspace-context | yes/no | complied / violated / N/A | graph orient? |
```

One row per always-on file. **violated** on procedural/hook tier = **HIGH** finding.

### 4.3 Plan completion (always third)

When no plan, todos, or approved-build scope exists in the thread: **`Plan: N/A`** in Action summary and this one-line block:

```markdown
## Plan completion

No plan, todo list, or approved-build scope in this thread.
```

When a plan exists, one row per item:

```markdown
## Plan completion

| Item | Status | Evidence |
|------|--------|----------|
| <plan step or todo id> | done (verified) / done (unverified) / partial / not started / N/A | <diff, oracle, or gap> |
```

Statuses:
- **done (verified)** — evidence in Verification ledger
- **done (unverified)** — claimed done; not re-checked this session
- **partial** — started but incomplete vs plan spec
- **not started** — no evidence of work
- **N/A** — out of scope for this audit

Any **not started** or **partial** row must appear in Action summary under **Plan: incomplete**.

### 4.4 Detail (after mandatory sections)

```markdown
## Findings

| Severity | Location | Finding | Evidence |
|----------|----------|---------|----------|
| HIGH | path or component | What is wrong / risky | verified / unverified / inferred: … |

Severity: **HIGH** (blocks production / data loss / security) · **MEDIUM** (fragile, drift, false confidence) · **LOW** (cosmetic, docs, nice-to-have)

## Themes
- Cross-cutting patterns (e.g. "documented != enforced")

## Prior audit delta
- New since last audit · unchanged · regressed
```

Omit empty Findings/Themes sections only when truly none. Do **not** omit §4.1–§4.3.

## 5. After the audit

- Offer to capture: if findings are durable, fire `capture_v2.py` with `scope: audit` (or ask Matt first for trivial scans)
- Optional: write report to `/Volumes/Cloud Storage/Memory/conversations/topics/<slug>-audit.md` **only if Matt asks to persist**

## Triggers (non-exhaustive)

**Audit-only:** `audit`, `/audit`, `audit your work`, `session audit`, `audit what you just did`, `assess`, `health check`, `review the system`, `what's broken`, `coverage audit`, `audit rules`, `audit hooks`, `audit memory`

**Plan verify loop:** `/verify-plan`, `audit and verify your plan`, `verify the plan`

**Audit + fix:** `/myauditandfix`, `audit and fix`, `fix any issues` (with audit context), `audit your work` + fix language in same message

## Anti-patterns

- Ending the turn after Phase 1 §4 report when fixable HIGH/MEDIUM remain (waiting for Matt to authorize Phase 2)
- Treating “report before fix writes” as a pause-for-ack checkpoint
- Treating audit as implicit approval to fix
- Mixing unrelated systems in one audit scope
- Findings or ledger rows without **verified** / **unverified** / **inferred**
- Skipping Action summary, Verification ledger, or Plan completion (including session audits)
- Auditing repo-wide `git diff` or pre-session WIP under `/myauditandfix` or session-audit triggers
- Stopping after one fix round when `/myauditandfix` was invoked and audit is not green (max 4 rounds per command)
- Declaring **Green: Y**, “no more HIGH,” or “all clear” immediately after Phase 2 **without** a post-fix full or `DELTA_CHECK` confirm Task
- Treating `NEW_HIGH_FROM_FIX: false` as green or as a substitute for re-audit
- Using confirm-only delta when the prior confirm had any HIGH (must full re-audit until a confirm returns zero HIGH)
- Using confirm-only delta when the fix introduced a new HIGH / HIGH regression
- Burying plan gaps only in Findings instead of Plan completion + Action summary
- Substituting bugbot-only output for the mandatory report sections
- Solo-auditing in orchestrator thread instead of dual critics + verifier
- Confirm+fix in a single `audit-verifier` Task dispatch
- Defaulting bugbot on session track without Matt opt-in
- Skipping parallel explore on multi-area system audits (serial read loops)
