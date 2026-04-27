# Pareto Frontier Subskill — Experiment Notes

**Date:** 2026-04-27
**Status:** Considered and rejected. Preserved on branch `experiment/pareto-frontier` for future reference; not merged to main.

## What was built

A `simmer:pareto-frontier` subskill that tracks non-dominated candidates across iterations and produces a frontier-context note for reflect. Files on this branch:

- `skills/pareto-frontier/SKILL.md` — the subskill (~280 lines)
- `skills/SKILL.md` — orchestrator wiring (Step 3.5 between judge and reflect, gated by `FRONTIER_MODE: on`)
- `skills/simmer-reflect/SKILL.md` — one bullet added: read frontier-context if present
- `CLAUDE.md` — design-decisions paragraph, subskill table row
- `tests/fixtures/pareto-frontier/` — 4 sequential test fixtures + verified standalone outputs
- `docs/simmer/vampire-arm-{a,b}-*/` — the A/B test run artifacts (3 iterations each)
- `docs/simmer/vampire-comparison.md` — initial comparison writeup (note: leans on cross-run composite scores, which the qualitative re-read corrected)

## Hypothesis (from the original spec)

GEPA-style Pareto frontier tracking would help simmer on tasks with criteria that genuinely trade off (architecture specs, schema design, DSL definitions). By preserving specialists across iterations — candidates strong on one dimension even if weaker overall — the frontier provides cross-iteration memory that linear simmer's most-recent-candidate-wins approach loses.

The frontier-context produced each round would name a specific transferable technique from a relevant specialist for the next generator.

## What the test showed (qualitative re-read)

A/B test on a vampire-mansion D&D module, three criteria intended to trade off (atmospheric_density, dm_runnable_clarity, player_agency), 3 iterations per arm, FRONTIER_MODE off vs on.

**Per-step quality was equivalent across both arms.** Both arms' ASIs were equally well-targeted. Both produced clean single-direction edits. Both reached comparable artifact ceilings via comparable per-step quality.

**Frontier collapsed to a single member after iter 1 in Arm B** (iter-2 strictly dominated iter-1; iter-3 strictly dominated iter-2). The multi-specialist mechanic — the actual reason the subskill exists — never engaged.

**Frontier-context's distinct contribution was redundant with the ASI on this run.** The iter-2 frontier-context pre-named two specific images (child's handprint, worn coffin lid) that the iter-3 generator copied near-verbatim, but the iter-2 ASI also named those exact images in its closing paragraph. The same transfer would have happened without the subskill.

## The actual reason it was rejected

The hypothesis assumes equal-weight criteria with stiff trade-offs and no clear user preference ordering. Real simmer users don't show up that way. They show up with a primary criterion (which simmer's setup brief already supports) and floors on the others. Once you have a preference order, there's no Pareto frontier — there's just constrained optimization, which the existing pipeline already handles.

The "trade-off-y" framing is mostly artificial:
- If a user cares about file size, they set it as the primary criterion or as a hard floor.
- If they care about latency, same.
- Hard structural choices (REST vs RPC, normalized vs denormalized) are picked before simmer runs, not surfaced as Pareto-incomparable iterations.

GEPA's actual setting — specialists across *sub-problems* in a benchmark distribution — doesn't map to simmer's setting of three judgments of the same artifact.

## What simmer already covers

- **Primary criterion + composite tiebreaker:** preference ordering. Already in the setup brief as `PRIMARY:`.
- **Stable wins / not-working tracking:** recognition memory of what worked and what didn't. Already in `simmer-reflect`'s output.
- **Best-so-far rollback on regression:** prevents the loop from building on a regressed state. Already in `simmer-reflect`.

Together, these handle the cases the frontier subskill was supposed to address.

## Where the frontier could still earn its keep (open question)

- A task where the user genuinely hasn't ranked their preferences and wants to *see* the frontier. That's closer to `test-kitchen:omakase-off`'s parallel-design exploration than to iterative refinement.
- GEPA-style optimization where the artifact is a prompt evaluated across a benchmark with sub-problem distribution. Specialists per sub-problem matter. That's a different skill entirely — not a simmer subskill.

If a future task surfaces either case clearly, revisit this branch.

## Why it's preserved on a branch instead of deleted outright

- The spec was thoughtful and the implementation is complete and tested. Cheap to preserve.
- The standalone subskill test (4-case fixture sweep) passed all spec §10 acceptance criteria — the code itself is correct, the issue is the use-case fit.
- If GEPA-style frontier tracking ever becomes load-bearing for some future simmer task, this branch is a working starting point.

## How to revisit

```
git checkout experiment/pareto-frontier
```

Branch contains the full implementation, tests, and the vampire A/B run artifacts. The qualitative comparison verdict above is the honest read; `docs/simmer/vampire-comparison.md` on this branch was written before that re-read and shouldn't be trusted for the conclusion.
