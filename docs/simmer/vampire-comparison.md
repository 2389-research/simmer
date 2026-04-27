# Pareto Frontier A/B Test — Vampire Mansion Module

**Task:** Generate a complete D&D 5e adventure module for level 3 PCs hired to slay a vampire in a Gothic mansion.

**Criteria (designed to trade off):**
- atmospheric_density
- dm_runnable_clarity
- player_agency

**Iterations per arm:** 3 (seedless)

## Headline result

| Arm | FRONTIER_MODE | Final composite | Δ vs seed |
|-----|---------------|-----------------|-----------|
| A   | off           | 8.3             | +0.6      |
| B   | on            | 9.3             | +1.3      |

**Arm B outperformed by 1.0 composite points.** Caveat below.

## Trajectories

**Arm A (clarity-led seed):**

| Iter | atm | clarity | agency | Composite |
|------|-----|---------|--------|-----------|
| 1    | 7   | 8       | 8      | 7.7 |
| 2    | 8   | 8       | 8      | 8.0 |
| 3    | 9   | 8       | 8      | 8.3 |

Movement on one axis (atmosphere). Linear, monotonic prose pass.

**Arm B (agency-led seed):**

| Iter | atm | clarity | agency | Composite |
|------|-----|---------|--------|-----------|
| 1    | 8   | 7       | 9      | 8.0 |
| 2    | 9   | 9       | 9      | 9.0 |
| 3    | 9   | 10      | 9      | 9.3 |

Movement on two axes; sharp +1.0 jump in iter 2, +0.3 in iter 3 (clarity hit ceiling).

## Frontier behavior in Arm B

- Iter 1 → frontier = [iter-1]
- Iter 2 → iter-2 dominated iter-1 (atm 8→9, clarity 7→9, agency 9=9). frontier = [iter-2]
- Iter 3 → iter-3 dominated iter-2 (clarity 9→10, others tied). frontier = [iter-3]

**The frontier collapsed to a single member every round.** No specialists were ever maintained in parallel. This is the spec §12 known risk — when criteria don't actually trade off in practice, the frontier collapses (correctly, that's its job) but the multi-specialist machinery never gets exercised.

## What's confounded in this comparison

Arm B's win is suggestive but not isolated:

1. **Different starting stance.** Arm A iter 1 was clarity-led (atm 7, clarity 8, agency 8). Arm B iter 1 was agency-led (atm 8, clarity 7, agency 9). Same composite ceiling at iter 1 (~7.7 vs 8.0) but different headroom — Arm B started with clarity at 7, leaving 3 points to climb; Arm A started at 8, leaving 2.
2. **Run-to-run generator variance.** Two seedless runs with different framing produced genuinely different artifacts at iter 1. Some of Arm B's score advantage is iter-1 luck, not later-iteration improvement.
3. **Frontier didn't earn its keep — exactly.** With only one frontier member at any time, the frontier-context's "Relevance to this iteration" pulled techniques from the *current* member rather than borrowing across specialists. Useful concrete suggestions came out (the iter-2 frontier-context proposed the child-handprint and worn-coffin-lid images verbatim, which the iter-3 generator copied as-is and they landed cleanly), but that level of transfer is achievable without Pareto tracking — a smart judge could produce the same suggestions in its ASI.

## What the test does and does not show

**Does show:**
- The wired pipeline works end-to-end. Generator → judge → frontier → reflect with `FRONTIER_MODE: on` produces valid state files and frontier-context notes through 3 iterations without error.
- The dominance check is correct. iter-2 vs iter-1 (dominates), iter-3 vs iter-2 (dominates) — both handled correctly with explicit side-by-side comparison and proper drop-log entries.
- The relevance section meets the §10 specificity criterion even with a single frontier member — concrete techniques and named passages, no vague "be more X" form.

**Does not show:**
- Whether Pareto tracking helps when criteria genuinely trade off. The vampire module turned out to be aligned-in-practice for these three criteria — a properly executed clarity uplift didn't have to sacrifice atmosphere or agency. Spec §11's instinct was right: D&D content is mostly aligned-criteria territory, even at full-module scale.
- Whether the score advantage is from the frontier subskill or just from Arm B's different starting stance. Single A/B run isn't enough signal to disentangle.

## Suggestions for a real test of the frontier machinery

To exercise the multi-specialist mechanic, the next test should pick a task where two valid candidates *cannot* both be made strong on the same criterion combo. Candidates:

1. **API spec design with a hard length cap** (e.g., "stay under 800 words"). Forces choices: extension surface vs terseness vs backwards-compat under a real budget constraint.
2. **DSL grammar definition with a parser-complexity cap.** Expressiveness vs parser simplicity vs error-message quality genuinely trade off.
3. **Schema design under storage/index trade-offs.** Normalization vs query speed vs migration safety can't all be max'd.
4. **Same vampire module but with a hard word cap (1500 words).** Atmosphere vs clarity vs agency become genuinely zero-sum at that budget.

Option 4 is the cheapest follow-up — same domain, same setup brief, just add `LENGTH_CAP: 1500 words` and rerun both arms.

## Final artifacts

- Arm A: `docs/simmer/vampire-arm-a-no-frontier/result.md` (clarity-led, 8.3/10)
- Arm B: `docs/simmer/vampire-arm-b-frontier/result.md` (agency-led with full clarity build-out, 9.3/10)

Both are usable modules. Arm B is the better one as a published artifact — more solutions to the climax, cleaner front-loaded combat reference card, the worn-coffin-lid detail is genuinely good. Arm A is leaner and runs in less prep time.
