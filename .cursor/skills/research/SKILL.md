---
name: research
description: >-
  Unified research router — AskQuestion for gather vs deep when mode is ambiguous;
  gather returns sources + short synthesis + conflicts (orchestrator relays).
  Triage quick/low→AskQuestion; single lookup→web-search. Deep: effort+profile
  AskQuestion then run.py. Tavily + Firecrawl + YouTube via scripts/_clients;
  X deferred. Prefer this over tavily-research / tavily-search plugin skills.
---

# Research

Orchestrator-owned **triage + dispatch**. LLMs do not browse independently in gather or deep — evidence comes from `scripts/_clients` (gather) or `web_context.py` inside `run.py` (deep).

**Not research:** single lookups → `web-search` skill. URL fetch → `web-fetch` skill.

**Do not load** official Tavily plugin skills (`tavily-research`, `tavily-search`, `tavily-extract`, …) when this harness is active — they use the `tvly` CLI and bypass gather/deep evidence, YouTube, and deep pipeline wiring. Disable the Tavily plugin in Settings → Plugins if the picker shows duplicates. See `/Volumes/Cloud Storage/Memory/conversations/topics/cursor-web-providers.md` § Plugin dedup.

## Triage table

| Matt says | Mode | Effort / preset | Dispatch |
|-----------|------|-----------------|----------|
| "what is / look up / find" (no research intent) | **inline** | — | `web-search` skill |
| "research X" (bare) | **AskQuestion mode** | — | mode popup → then gather or deep |
| "research X, quick / low effort" | **AskQuestion mode** | `quick` (if Deep chosen) | effort-only does **not** skip mode popup |
| "comprehensive / report / literature review / scholarly" | **deep** | balanced + format | `Task(research, mode=deep, run_in_background=true)` |
| "high / deep dive / aggressive" | **deep** | `aggressive` | `--preset aggressive` |
| "latest news on X" | **gather** | — | Tavily news + recency in gather (no mode popup) |
| "video / tutorial / talk / YouTube" | **gather** | — | YouTube prioritized + web providers (no mode popup) |
| explicit `gather` / `deep` | named mode | as applicable | dispatch without mode popup |
| Deep triggered, effort unspecified | **deep** | **balanced** | `--preset balanced` |

**Effort mapping:** low → `quick`, medium → `balanced`, high → `aggressive`.

## § Mode preflight — AskQuestion (ambiguous only)

**Fire when:** intent is research-agent dispatch and mode is not implied (bare `"research X"`, or effort-only like quick/low with no report/deep cues).

**Skip when any of:** explicit gather/deep/comprehensive/report/literature review/scholarly/deep dive; single lookup → `web-search`; strong gather cues (`latest news`, video/tutorial/YouTube); Matt already named mode in-thread this turn.

**Popup options (Recommended = Gather):**

| Option | Meaning |
|--------|---------|
| **Gather (Recommended)** | multi-source evidence + short synthesis + conflicts; foreground; max 5 paid calls |
| **Deep** | full cited report with verifier pipeline; background; then effort + profile AskQuestions below |

After Gather is chosen, agent returns sources + short synth + conflicts; **orchestrator relays** to Matt (Flavor-ON).

## Deep preflight — AskQuestion (required)

Before `Task(research, mode=deep, run_in_background=true)`, run **AskQuestion** when **either** effort **or** model profile is missing from the prompt.

**Skip** deep effort/profile AskQuestion when both are explicit, e.g. `"comprehensive report, high effort, default models"` or `"deep research X, medium, grok-only"`.

### Question 1 — Effort

| Option | `--preset` |
|--------|------------|
| Low | `quick` |
| **Medium (Recommended)** | `balanced` |
| High | `aggressive` |

### Question 2 — Model roster

Web evidence is always Tavily + Firecrawl + YouTube (not selectable). LLM roles only:

| Profile ID | Label | Researchers | Synthesizer | Adjudicator |
|------------|-------|-------------|-------------|-------------|
| `default` | **Default (Recommended)** — mixed pool, Grok synth | grok-4.3, deepseek-chat, MiniMax-M3 | grok-4.3 | grok-4.3 |
| `grok-only` | All Grok — fastest, single provider | grok-4.3 ×3 | grok-4.3 | grok-4.3 |
| `mixed-economy` | Grok + DeepSeek — no MiniMax | grok-4.3, deepseek-chat | grok-4.3 | grok-4.3 |

Profiles: `scripts/deep_research/profiles/{id}.yaml`. Pass `--profile <id>` to `run.py`.

### Optional local persistence (v1)

Gitignored `scripts/deep_research/deep_research_config.local.yaml` merges over profile + `config.yaml` when present. Use to persist last AskQuestion choices locally; pre-select as Recommended on next AskQuestion (orchestrator may read file if it exists — do not commit).

## Format knobs (deep)

Pass through to `run.py`: `--format`, `--citation-style`, `--sections`, `--visual-level`.

Supported formats: `briefing_memo`, `literature_review`, `pros_cons`, `how_to`, `scholarly`.

## Dispatch

```
Task(subagent_type: "research", readonly: true, run_in_background: <true only for deep>)
```

Prompt must include: `mode`, `query`, and for deep: `preset`, `profile` (post-AskQuestion).

**Gather:** foreground, max 5 paid provider calls; agent returns sources + short synthesis + conflicts; **orchestrator relays** Lane A to Matt.

**Deep:** background; relay `outputs/research/<slug>.md` excerpt Lane B.

## YouTube cues

- YouTube URL in query → metadata via `videos.list` (1 quota unit), skip search
- Video/tutorial intent → `search_videos` once per gather (100 units); transcripts for top 5 (free)
- Deep pipeline includes YouTube in `web_context.py` when relevant

## X / Twitter (deferred)

`x_call.py` returns `{"error": "x_not_configured"}` with no network. Note in gather evidence if queried; do not retry or block on X.

## Related skills

| Skill | When |
|-------|------|
| `web-search` | Single lookup, no multi-source research |
| `web-fetch` | One URL extract/scrape |

## Oracle

```bash
bash scripts/smoke_web_providers.sh
bash scripts/smoke_deep_research.sh
bash .cursor/scripts/smoke_harness_agents.sh
bash .cursor/scripts/smoke_harness_skills.sh
```
