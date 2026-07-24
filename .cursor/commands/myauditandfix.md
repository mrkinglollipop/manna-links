---
name: myauditandfix
description: Session audit (this thread) with mandatory report, then loop audit→fix until green (max 4 rounds)
---

# Session audit and fix

**Authoritative contract:** `.cursor/skills/audit/SKILL.md` and `.cursor/rules/audit.mdc`. This command **authorizes the fix phase** — equivalent to "audit your work and fix any issues."

**Pipeline:** orchestrator **Freshness pass** (when applicable) → dual `session-auditor` (`TRACK=session`, bug_hunt + claim_bust) → confirm-only `audit-verifier` (`TRACK=session`) → mandatory §4 report to Matt → fix `audit-verifier` when not green. Orchestrator remains hub; does not solo-audit instead of dual critics.

## Scope

- **Default:** session audit — work done **only in this thread** (the chat where `/myauditandfix` was invoked):
  1. **Conversation writes** — paths touched by `Write`, `StrReplace`, or `Delete` in this thread (orchestrator + subagents).
  2. **Session side effects** — paths changed by shell in this thread (e.g. `capture_v2.py` → `episodes.jsonl` + any topic pages it wrote).
  3. **Load-bearing chat claims**, plan/todos, and rule-compliance (dispatch, build gate, read-only discipline, **Deliverable Contract compliance + intent alignment vs first user build request**) **from this thread**.
- **Not in scope:** pre-session uncommitted WIP — do **not** treat repo-wide `git diff` as the session audit surface. If a path was not touched in this thread, it is out of scope unless trailing text names it explicitly.
- **Narrow:** trailing text after `/myauditandfix` narrows or adds a target (path, subsystem, constraint). Still run Phase 2 unless trailing text explicitly says audit-only / no fixes.
- **Out of scope:** unrelated workspace areas, drive-by refactors, new features beyond fixing listed findings.

**Depth:** thorough. **Track:** **`TRACK=session`** on every critic and verifier dispatch (§0 session audit in SKILL). Skip graph orient (§1) when target is only this thread's local files.

**Bugbot:** default **off** for session track. Opt in only if Matt's prompt/trailing text asks for it, or (rare) when the session is clearly security-sensitive and Matt did not forbid it.

## Artifact pack (orchestrator → critics + verifier)

Every dispatch for a round must include:
- Scope block (target, in/out, depth)
- **`TRACK=session`**
- Session file set (paths from this thread's Write/StrReplace/Delete + named shell side effects)
- Load-bearing chat claims list (verbatim or quoted)
- Plan/todo ids in thread (or "none")
- Prior round finding deltas (rounds 2–4)
- When `DELTA_CHECK=true`: paths touched by last Phase 2 fix, prior confirmed findings + clearance claims, fix-round oracle tails (critic reports optional — omit dual critics)
- Oracle log tails already collected this round (if any)
- **Freshness oracle notes** (orchestrator-collected before critics when Freshness pass ran; use "none" otherwise)
- For verifier step 1: **both critic reports** (bug_hunt + claim_bust) — **unless** `DELTA_CHECK=true`
- For verifier step 2 only: the confirmed finding list from step 1 (no re-litigating dropped items without new evidence)

**Pin Task `model` on every audit dispatch:** critics → `model: "composer-2.5-fast"` + `readonly: true` (omit denied). Adjudicator (confirm + fix) → **only** **omit** `model` or `cursor-grok-4.5-high`. Never k3 as an adjudicator pin. Never Composer. **Default: omit** (frontmatter Grok-high backs omit under Auto). Optional pin high ONLY when a prior Task in **this conversation transcript** landed with explicit `model: cursor-grok-4.5-high` without product deny / "Couldn't start"; if unsure → omit; never-first-under-Auto. On pin deny: retry omit once; never k3; never Composer. If omit fails → STOP. Escape hatch if Task enum lacks subagent_type: `generalPurpose` / `explore` / missing-type + read `.cursor/agents/<name>.md` with the **same** rules. Never bare `grok-4.5-high` or Grok `*-fast`. Native types preferred.

## Freshness pass (orchestrator — before dual critics when applicable)

Run when session claims cite paths, versions, APIs, or external "current" facts.

**Owner:** orchestrator collects notes **before** dual critics; critics may add incidental free checks only.

1. **Always (free):** existence/help oracles for cited session paths/commands; compare claims to current files.
2. **When claims need prior context (free):** graph `search` + memory topics grep — not a full-workspace orient every round.
3. **When applicable (paid, ≤5/round, orchestrator budget):** versioned external surfaces → `.cursor/skills/web-search/SKILL.md` / `scripts/_clients`. Cite URL + date in ledger.

Put results in artifact pack as **Freshness oracle notes**. Paid web budget does **not** consume each critic's 3 shell/oracle-run cap.

**Delta freshness reuse:** Freshness pass before dual critics when running **full** re-audit; `DELTA_CHECK=true` rounds may reuse last freshness notes unless session paths/claims changed.

## Phase 1 — Audit (read-only)

Do **not** edit files during this phase.

1. State scope block: target, in scope, out of scope, depth.
2. Run **Freshness pass** when applicable; attach **Freshness oracle notes** to artifact pack.
3. **Dual critics (parallel):** dispatch two `session-auditor` Tasks in one turn with **`TRACK=session`**, **`model: "composer-2.5-fast"`**, `readonly: true`:
   - `ROLE=bug_hunt` — correctness, regressions, edge cases, silent failures
   - `ROLE=claim_bust` — chat claims vs evidence, false done, process gaps, **shared freshness items** (stale/false paths, outdated API/version claims, chat "verified" without evidence, scope creep vs thread intent, freshness failures)
4. **Confirm-only verifier:** dispatch `Task(subagent_type: "audit-verifier", readonly: true)` with `fix_authorized=false`, **`TRACK=session`**, both critic reports — **unless** `DELTA_CHECK=true` — and Freshness oracle notes in the artifact pack. Adjudicator model: omit-first (omit default; optional `cursor-grok-4.5-high` only if this transcript already saw Task accept that pin; on deny retry omit once; never k3 as an adjudicator pin; never Composer). Dual-mode agent cannot use frontmatter `readonly`; dispatch-level `readonly: true` is the mechanical backstop for confirm-only. Verifier confirms/rejects/dedupes and returns §4-ready payload.
5. **Optional bugbot:** only when Matt opted in — fold into Findings; does not replace §4.1–§4.3.
6. Orchestrator surfaces **mandatory report** to Matt in order (from verifier payload + synthesis):
   - **Action summary** (verdict, do now, blocked on Matt, plan status)
   - **Verification ledger** (every row: verified / unverified / inferred)
   - **Plan completion** (row per plan/todo item, or N/A)
   - **Findings** (severity table + themes)

**Two-step rule (HARD):** mandatory §4 report is surfaced to Matt **before any fix writes**. Verifier never confirm+write in a single Task.

**Same-turn continue (HARD):** After the §4 report is surfaced in the orchestrator message, if the audit is **not green** and **not stop-early**, **immediately** dispatch Phase 2 (`fix_authorized=true`) in the **same turn** — do **not** wait for Matt acknowledgment, “continue”, or “go fix”. `/myauditandfix` / `/verify-plan` already authorize the fix phase. Two-step only constrains **order** (report text before fix writes) and forbids confirm+fix in one Task — it is **not** a human checkpoint between phases.

## Phase 2 — Fix (authorized by this command)

1. Dispatch `audit-verifier` with `fix_authorized=true`, **`TRACK=session`**, adjudicator model omit-first (omit default; optional `cursor-grok-4.5-high` only if this transcript already saw Task accept that pin; on deny retry omit once; never k3 as an adjudicator pin; never Composer), and the **confirmed finding list** from Phase 1 step 1 — fix **only** those findings; no scope creep.
2. Orchestrator reviews diff + runs oracle gate. No LOC force-dispatch.
3. Re-verify each fix via oracle/ledger updates only — **this is not a re-audit**. Fix-agent self-report (`NEW_HIGH_FROM_FIX`, clearance notes) is **input to post-fix mode selection only**; it does **not** authorize **Green: Y**, “zero HIGH/MEDIUM,” or ending the loop.

## Phase 3 — Loop until green (mandatory for `/myauditandfix`)

Run **sequentially** — do not start the next round until the current round's Phase 2 is done **when Phase 2 ran**. Exit without Phase 2 when already **green** or **stop-early** after confirm.

**Green** (stop looping) when **all** of:
- Findings: **zero** HIGH and **zero** MEDIUM rows — as of the **latest confirm** (full confirm or `DELTA_CHECK` confirm), **not** as of the fix verifier's self-report
- Plan completion: no **partial** or **not started** rows for in-scope items
- Every load-bearing claim in Verification ledger is **verified** or explicitly **blocked on Matt** (not silently **unverified**) — **freshness-verified ledger gate**
- Action summary **Do now** is "Nothing blocking — ship" (or equivalent)

**Not green** (keep looping) when any **fixable-in-session** HIGH/MEDIUM finding remains, any in-scope plan item is partial/not started, load-bearing freshness claims lack verification or explicit Matt blocker, or fixes introduced new regressions surfaced in re-audit.

**Stop early — Blocked on Matt (HARD):** after confirm (any round), if every remaining HIGH/MEDIUM finding is explicitly **Blocked on Matt** / requires Matt authorization (credentials, commit/push, taste, irreversible outward action, or a decision only Matt can make) **and** there are **zero** fixable-in-session HIGH/MEDIUM findings — **exit the loop immediately**. Do **not** start Phase 2 or further re-audit rounds. Surface the §4 report (or Re-audit block), list blockers under **Blocked on you**, and end with **Loop summary** `**Green:** N` (not Y — Matt action still required). Wasted rounds that cannot clear without Matt are forbidden.

**Round cap:** **4 rounds** total (round 1 = first Phase 1 + optional Phase 2; rounds 2–4 = re-audit/delta + fix). After round 4, stop even if not green; report remaining findings and **Blocked on you** items. Stop-early (above) may exit sooner.

**Post-fix re-audit mandatory (HARD):** after **every** Phase 2 that ran, you **must** start the next round's audit (full or delta) **before** any claim of green, “no more HIGH,” “cleared,” or Loop summary — **in the same turn**; do **not** end the turn on fix alone or wait for Matt before re-audit. `NEW_HIGH_FROM_FIX` and whether the prior confirm had any HIGH choose shape only — never skip re-audit / never equals green.

**Post-fix re-audit mode (HARD):** after each Phase 2, choose the next round's audit shape before dispatching:
1. **Full re-audit** when **any** of:
   - the **prior confirm** (full or delta) had **any HIGH** finding — including HIGH the fix claims to have cleared (clearing prior HIGH requires dual critics, not delta); **or**
   - the fix introduced **any new HIGH** (including a previously-cleared HIGH reopened as HIGH), or the fix verifier / oracle gate reports a HIGH regression  
   → Freshness pass (when applicable) + dual `session-auditor` + confirm-only verifier.
2. **Confirm-only delta check** only when **both**:
   - the **prior confirm** had **zero HIGH**, **and**
   - the fix introduced **no new HIGH** (`NEW_HIGH_FROM_FIX=false`)  
   → **skip dual critics**. Dispatch one confirm-only `audit-verifier` (`fix_authorized=false`, adjudicator model omit-first (omit default; optional `cursor-grok-4.5-high` only if this transcript already saw Task accept that pin; on deny retry omit once; never k3 as an adjudicator pin; never Composer; still `readonly: true`)) with **`DELTA_CHECK=true`**, fix-touched paths, prior confirmed findings + clearance claims, and fix-round oracle tails. New MEDIUM-only issues do **not** force full re-audit. Delta confirm is still a **real second pass** — clearances are not green until this Task returns zero HIGH/MEDIUM (or stop-early).
3. If a confirm-only delta check **surfaces a new HIGH**, do **not** Phase 2 yet — **escalate in the same round** to full re-audit (dual critics + confirm) before further fixes. Still counts as one round toward the cap.

**Per round:**
1. **Round 1:** full mandatory report (§4.1–§4.4) before any edits (always dual critics + confirm). Apply stop-early if only Matt blockers remain.
2. **After Phase 2:** always open rounds 2–4 via post-fix re-audit mode (never end on fix alone). Emit **Re-audit (round N/4)** or **Re-audit (round N/4) — delta**. Findings / ledger / Plan completion **deltas only**; skip unchanged rows. Apply stop-early after confirm (or after escalate-confirm).
3. If not green, not stop-early, and round `< 4`: Phase 2 (fix verifier) on confirmed findings only, then **mandatory** next re-audit round.
4. If green or stop-early **after a confirm/delta confirm**, or round `= 4`: exit loop.

**End state:** short **Loop summary** that includes a string matching `.cursor/hooks/audit_marker.py` `GREEN_RE` when green so the push-audit stamp can fire — prefer `**Green:** Y` or `Green: Y` (also accepted: `rounds used … Green: Y` / `Loop summary … Green: Y`). When not green, use `**Green:** N` or `Green: N`. **Push OK also requires dual-critic→confirm Task dispatch evidence in the transcript** (`tool_use` Task blocks, not assistant prose alone) when `/myauditandfix` was invoked (see `audit_marker.py` pipeline gate; kill switch `/tmp/.cursor_audit_pipeline_gate_disable`). Also include: rounds used (e.g. 2/4), re-audit mode used each round (full vs delta), what changed across rounds, what was re-verified vs still unverified, any findings left after cap. **Green: Y requires at least one post-fix confirm or delta-confirm Task in the transcript when Phase 2 ran** (fix self-report alone is insufficient).

**After Green: Y — stamp_ok (cloud / stop-miss):** when this conversation may lack a local agent transcript (Cloud Agent `bc-*` ids, or stop hook did not stamp), run before any `/commitprmerge` push. With a real `conversation_id`, `stamp_ok` requires a prior PENDING marker (`/myauditandfix` or `/verify-plan` already invoked); without PENDING it no-ops. Workspace-only stamp (no `conversation_id`) can still mint WS_OK without PENDING by design.

```bash
python3 "/Volumes/Cloud Storage/Claude/.cursor/hooks/audit_marker.py" stamp_ok <<EOF
{"conversation_id":"<id if known>","cwd":"$(pwd)","workspace_roots":["$(pwd)"]}
EOF
```

This is the authorized recovery path — not inventing bypass files. Local chats with a real transcript still rely on stop + pipeline evidence; `stamp_ok` is backup when that path cannot see the transcript.

## Anti-patterns (do not)

- Ending the turn after Phase 1 §4 report when fixable HIGH/MEDIUM remain (waiting for Matt to authorize Phase 2)
- Treating “report before fix writes” as a pause-for-ack checkpoint
- Patching before the mandatory report sections exist
- Confirm+fix in a single `audit-verifier` Task dispatch
- Solo-auditing in the orchestrator thread instead of dual critics + verifier
- Treating this as implicit `approved — build` for new features or broad multi-file work
- Substituting bugbot-only output for Action summary / Verification ledger / Plan completion
- Defaulting bugbot on session track without Matt opt-in
- Auditing repo-wide `git diff` or pre-session WIP instead of this thread's file set
- Stopping after one fix round when `/myauditandfix` was invoked and audit is not green (max 4 rounds per command)
- Declaring **Green: Y**, “no more HIGH,” or “all clear” immediately after Phase 2 **without** a post-fix full or `DELTA_CHECK` confirm Task
- Treating `NEW_HIGH_FROM_FIX: false` as green or as a substitute for re-audit
- Using confirm-only delta when the prior confirm had any HIGH (must full re-audit until a confirm returns zero HIGH)
- Using confirm-only delta when the fix introduced a new HIGH / HIGH regression (must full re-audit)
