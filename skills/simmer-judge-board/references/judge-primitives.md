### Judge Primitive Library

Building blocks for constructing judges. Apply relevant ones to each judge based on their role.

**Core (apply to all judges):**
- Score via seed calibration — score the original first, anchor all subsequent iterations to it
- Diagnose before scoring — read the candidate, evaluator output, and relevant code/config. Understand *why* things are the way they are before writing scores.
- Protect high-scoring elements — identify what's working and constrain your ASI to preserve it
- Score ALL criteria from your lens — every judge scores every criterion from their perspective, not one criterion per judge

**When evaluator is present:**
- Cluster evaluator failures by type — near-misses (spelling), systematic gaps (whole category), noise (hallucinations). The pattern determines the fix.
- Verify proper nouns from lossy sources — transcripts, OCR, and auto-captions garble names
- Flag evaluator stochasticity — if the same config produces different results, small score changes may be noise

**When the problem involves exploration:**
- Review what's been tried — check iteration history before suggesting more of the same
- Flag ceilings — if 2+ iterations tried the same type of change with no improvement, the bottleneck is structural
- Research if stuck — look up how similar problems are solved rather than guessing

### Example Compositions

These show how judges are constructed for different problems — they're examples, not templates.

**Code/pipeline with evaluator:**
| Judge | Why this lens | Key primitives |
|-------|--------------|----------------|
| **Evaluator Analyst** | Someone needs to deep-dive the metrics — what patterns emerge in pass/fail, where are the near-misses vs systematic gaps? | Cluster failures, flag stochasticity |
| **Constraint Realist** | The execution environment has specific capabilities and limits that affect what approaches work | Diagnose before scoring, flag ceilings, research if stuck |
| **Downstream User** | Does the output actually work for its intended use? Would a consumer of this output be satisfied? | Protect high-scoring elements, score via seed calibration |

**Creative writing (judge-only, no evaluator):**
| Judge | Why this lens | Key primitives |
|-------|--------------|----------------|
| **Craft** | Is the writing working at a technical level — structure, pacing, voice? | Diagnose before scoring, protect high-scoring elements |
| **Reader** | Does this land emotionally for someone reading it cold? | Score via seed calibration |
| **Domain Expert** | Does it get the genre/setting/rules right? | Research if stuck (for genre conventions) |

**Pipeline optimization (workspace, multi-model):**
| Judge | Why this lens | Key primitives |
|-------|--------------|----------------|
| **Metrics** | What do the evaluator numbers actually show? | Cluster failures, flag stochasticity |
| **Architecture** | Is the pipeline structure right, or is it a local optimum? | Flag ceilings, review what's been tried |
| **Operations** | Can this run reliably in production at acceptable cost? | Protect high-scoring elements |

### Custom Judge Panels

Users can define custom judges in the setup brief:

```
JUDGE_PANEL:
  - name: Technical Accuracy
    lens: Score with focus on factual correctness and logical consistency
  - name: Audience Fit
    lens: Score as a first-time reader with no domain background
  - name: Actionability
    lens: Score with focus on whether the reader can act on this immediately
```

Custom panels override auto-composition entirely. Minimum 2 judges, maximum 5. Default is 3.
