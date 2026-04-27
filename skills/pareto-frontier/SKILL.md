---
name: pareto-frontier
description: >
  Pareto frontier subskill for simmer. Maintains a set of non-dominated
  candidates across iterations and produces a frontier-context note that
  reflect can use to surface specialist strengths from prior rounds.
  Useful for tasks with criteria that genuinely trade off (architecture,
  schema, DSL design). Do not invoke directly — called by simmer
  orchestrator after each judge round, before reflect.
---

# Simmer Pareto Frontier

You maintain a Pareto frontier of simmer iteration candidates. After each judge round, you (1) update the frontier state, (2) log dominated candidates to a failure memory, and (3) produce a context note for reflect that names a specific transferable technique.

## Why This Exists

Linear simmer always carries forward the most recent candidate. That works when criteria align (clarity AND specificity AND tone all want to go up). It loses information when criteria genuinely trade off — a candidate that won big on extensibility but lost on terseness will be discarded after one regression, even though its extensibility move was right.

The frontier preserves specialists. If iteration 3 is the best on agency and iteration 5 is the best on specificity, both stay. Reflect gets to point the next generator at the right specialist's technique for the criterion the current candidate is weakest on.

## Context You Receive

- `iteration_n` — current iteration number
- `candidate` — either a path to `iteration-N-candidate.md` (single-file) or a workspace directory path (workspace mode); may also be passed inline as text
- `judgment` — judge output for this iteration. Either a path to a judgment file OR the judge's output passed inline (scores per criterion + per-criterion evidence + ASI). If passed inline, treat the inline content as the judgment and store `judgment_path: null` in the frontier entry.
- `frontier_path` — `{output_dir}/frontier.json` (may not exist yet)
- `dropped_path` — `{output_dir}/dropped.json` (may not exist yet)
- `criteria` — ordered list of criterion names
- `frontier_context_path` — output path for `iteration-N-frontier-context.md`
- `artifact_type` — `single-file` or `workspace`

## What To Do

### 1. Read state

Read `frontier.json` if it exists. If not, initialize:

```json
{"criteria": [<criterion names>], "members": []}
```

Read `dropped.json` if it exists. If not, initialize `{"entries": []}`.

### 2. Parse the new candidate

Read the candidate.

- **Single-file:** read the candidate file (or use the inline text).
- **Workspace:** read the key files the judge cited in their evidence. You don't need to read every file — read the ones that the judge's per-criterion evidence points at. If the judge cites file paths, follow them.

Read the judgment. Extract:
- per-criterion numeric score (1-10)
- per-criterion evidence text (the judge's reasoning for the score, both positive and negative)
- the ASI for this round

If the judgment is passed as a path, read the file. If passed inline, use the inline content.

### 3. Dominance check (do this carefully)

**Definition.** Candidate X dominates candidate Y iff:
- `X.score[c] >= Y.score[c]` for every criterion c, AND
- `X.score[c] > Y.score[c]` for at least one criterion c.

**Procedure.** For each existing frontier member, write the comparison out explicitly before drawing a conclusion. Format:

```
new (iter-5):  agency=8  specificity=9  terseness=6
existing iter-3: agency=9  specificity=6  terseness=7
  agency:      iter-3 > iter-5  → iter-3 not dominated by iter-5 on this dim
  specificity: iter-5 > iter-3  → iter-5 not dominated by iter-3 on this dim
  terseness:   iter-3 > iter-5  → iter-5 not dominated by iter-3 on this dim
  Conclusion:  Pareto-incomparable. Both stay.
```

Write this comparison block in your scratch reasoning before deciding. Skipping it produces arithmetic drift, especially with 4+ criteria.

**Outcomes:**

- **New candidate is dominated by any existing member** → drop the new candidate. Append a dropped entry. Do not add it to the frontier. Skip to step 6.
- **New candidate dominates one or more existing members** → drop those members from the frontier. Append a dropped entry for each, with `dominated_by: ["iter-N"]` (the new candidate's id). Add the new candidate.
- **Pareto-incomparable with all existing members** → add the new candidate. No drops.

A candidate with scores tied to an existing member on every criterion is *not* dominating (no strict-greater dim). Treat ties as Pareto-incomparable — keep both unless something else dominates one.

### 4. Update the frontier

Build the new entry (when adding):

```json
{
  "id": "iter-N",
  "candidate_path": "<path or directory>",
  "judgment_path": "<path or null if judgment was inline>",
  "scores": {<criterion>: <score>, ...},
  "best_at": [<criteria where this is now top among frontier members>],
  "key_passages": [
    {
      "criterion": "<criterion>",
      "excerpt": "<verbatim passage from candidate that drove the score>",
      "judge_evidence": "<verbatim sentences from judge explaining why>"
    }
  ],
  "weaknesses": [
    {
      "criterion": "<lowest-scoring criterion>",
      "judge_evidence": "<verbatim judge critique>"
    }
  ],
  "added_at_iteration": N
}
```

**`best_at` must be recomputed for ALL members after any change** — adding a member or dropping one. For each criterion, find the member(s) with the highest score on that criterion. Tie → all top scorers list it in `best_at`. A member can have an empty `best_at` (a balanced generalist that's not top on any single dim but isn't dominated either). Edge case: when there is only one member (iteration 1), that member is trivially top on every criterion — `best_at` lists all of them.

**`key_passages`:**
- For each criterion in this candidate's `best_at`, include one entry with a verbatim excerpt from the candidate (single-file) or a verbatim file:line-range excerpt (workspace).
- If `best_at` is empty, include one entry for the candidate's highest-scoring criterion anyway — the relevance section will need it.
- Verbatim means verbatim. Do not paraphrase. The point is downstream agents read the actual operative text, not a re-derived summary.

**`weaknesses`:**
- One entry for the candidate's lowest-scoring criterion.
- `judge_evidence` is the judge's verbatim critique of that criterion.

### 5. Update dropped log

For each dropped entry (whether the new candidate was dropped or it displaced existing members):

```json
{
  "id": "iter-N",
  "candidate_path": "<path>",
  "scores": {...},
  "dominated_by": ["iter-X", "iter-Y"],
  "failure_summary": "<one-liner extracted from judge's lowest-scoring evidence>",
  "dropped_at_iteration": <current iteration>
}
```

`failure_summary` is one short sentence. It's recognition memory, not study material. Aim for 8-15 words naming the move that didn't work, not a critique. Example: "Collapsed scenes 2 and 3, lost narrative escalation between acts."

If the judge cited multiple distinct failures, name the single move with the highest cost — the one driving the lowest score. Don't try to enumerate.

### 6. Write state files

Write the updated `frontier.json` and `dropped.json`.

### 7. Produce frontier-context.md

This is the file reflect reads. Use this template:

```markdown
# Frontier Context for Iteration {N}

## Current iteration's failure modes
[1-3 bullets naming criteria that scored low this round and the judge's
stated reason. Pull from the judgment's evidence section.]

## Frontier members and their strengths

### {member.id} — best on: {member.best_at joined with ", " | "balanced (no single-criterion win)"}
**Score profile:** {criterion}: {score} for each criterion in canonical order

**Operative passage:**
> {key_passage.excerpt}

**Why it works (judge evidence):**
> {key_passage.judge_evidence}

**Known weakness:**
> {weakness.judge_evidence}

---

[repeat for each frontier member]

## Relevance to this iteration

[The analytical section. See guidance below.]

## Recently dropped — avoid these failure modes
[Up to last 5 dropped entries, most recent first:]
- {entry.id}: {entry.failure_summary}
```

### The Relevance section is where this skill earns its keep

Procedure:

1. Identify the criterion (or criteria) where the current iteration's candidate scored lowest.
2. Find the frontier member(s) whose `best_at` includes that criterion (or whose score on it is highest).
3. Read those members' `key_passages` and `judge_evidence`.
4. Articulate a SPECIFIC transferable technique. Not "be more X" but "use approach Y from member Z, applied to context W of the current parent."
5. Note constraints — what strengths of the current parent must be preserved while applying the borrowed technique.

Good (concrete, names a move):
> "This iteration scored 4 on specificity. iter-5 scored 9 on specificity through sensory grounding in NPC introductions — see the operative passage above. The technique is portable: anchor each NPC reveal on a specific sensory detail (smell, posture, voice tic). The current parent is strong on agency (8) so the work is adding sensory grounding without disrupting the existing branching structure."

Bad (vague, redirects work to the next agent):
> "iter-5 is good at specificity. Try to be more specific."

If you find yourself writing the bad form, stop and re-read the operative passages. The technique should be concrete enough that the next generator could apply it without re-reading any other files.

If only one frontier member exists (e.g., on iteration 1), the Relevance section can simply summarize that member's strengths and weaknesses — no cross-member comparison is possible yet.

## Output to Orchestrator

```
FRONTIER UPDATED
MEMBERS: [list of iter-N ids]
ADDED THIS ROUND: [iter-N or "none — dominated"]
DROPPED THIS ROUND: [list or "none"]
FRONTIER_CONTEXT: <path to iteration-N-frontier-context.md>
```

The orchestrator passes the frontier-context path to reflect. Reflect reads it alongside the judgment.

## Common Mistakes

**Skipping the side-by-side comparison block**
- Problem: LLM arithmetic drift on dominance with 4+ criteria. Wrong member dropped or kept.
- Fix: Write the explicit per-criterion comparison in your scratch reasoning before concluding. Step 3 shows the format.

**Paraphrasing the operative passage**
- Problem: Downstream agents have to re-derive what the technique was, often guess wrong.
- Fix: Verbatim excerpts. If the passage is long, quote it long. Tokens are cheap.

**Vague Relevance section**
- Problem: Reflect can't act on "be more specific." The skill provides no value over linear simmer.
- Fix: Name a move from a specific member's operative passage. If you can't, the operative passage you stored isn't operative — go re-read the judge's evidence and pick a better excerpt.

**Forgetting to recompute `best_at` for all members**
- Problem: A new top scorer on criterion C is added but the previous top still claims `best_at: [C]`. Frontier becomes inconsistent.
- Fix: After any add or drop, recompute `best_at` for every remaining member, even ones not touched this round.

**Treating ties as dominance**
- Problem: Two candidates with identical scores both get marked dominated, frontier collapses.
- Fix: Dominance requires strict-greater on at least one dim. All-equal is Pareto-incomparable; keep both.

**Storing a redundant entry on a dominated candidate**
- Problem: New candidate is dominated by existing member but gets added anyway, frontier grows unboundedly.
- Fix: If the new candidate is dominated, append to `dropped.json` only — do NOT add to frontier members.

**Reverting `best_at` to a member that was just dropped**
- Problem: Member was dropped this round but still appears in another's recomputation lookup.
- Fix: Recompute `best_at` against the post-update members list, not the pre-update list.

## Red Flags — Stop and Re-Check

- You wrote dominance conclusions without the side-by-side block → re-do step 3.
- The Relevance section uses words like "more", "better", "improve" without naming a specific passage → re-do step 7's Relevance procedure.
- Two members both list the same criterion in `best_at` but have different scores on it → recompute, only top scorers (including ties) qualify.
- A dropped candidate appears in `frontier.json` members list → remove it.
- The new candidate has scores identical to an existing member and you dropped one → ties are not dominance; both stay.
