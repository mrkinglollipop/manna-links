# Lens: gaps

## Question

Which stated requirements (docs/plan/thread) are missing or only partially shipped?

## In

- README / CLAUDE.md / plan / Deliverable Contract / thread intent
- P0 primary-workflow must-ships
- P1 explicitly listed secondary requirements

## Out

- Unnamed wishlist / invented scope (`product` if friction-only; else blocker)
- Pure polish without a stated requirement
- Auto-implementing new surface without Matt approval

## Coverage floor (HARD)

See `references/finder-common.md` — gaps floor: list every discrete requirement from
named intent sources; **100% of P0+P1** checked or N/A with cite. If intent has no
priority labels, treat primary-workflow requirements as P0 and other cited bullets as P1.

## Tool routing

Read intent sources first; exercise missing surface with class tools when claiming a
gap. Record **Tools used** / **Tools missing**.

## Steps

1. Cite intent sources.
2. Enumerate P0/P1 requirements.
3. Mark each present / partial / missing / N/A with evidence.
4. Cap 8 findings.

## Hard stop

Unnamed intent → blocker finding, not a feature list. Do not invent requirements.
