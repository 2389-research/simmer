# Iteration 1 Judgment

## Scores

- backwards_compat: 7
- extensibility: 5
- terseness: 8

Composite: 6.7/10

## Per-criterion evidence

### backwards_compat (7)
Reuses the existing `/v1/auth/login` endpoint without changes — good. Path-prefix versioning under `/v1/` is the conventional choice and won't surprise existing consumers. Slight knock: no explicit deprecation header pattern documented, so future deprecations will be ad hoc.

### extensibility (5)
The Note resource is closed — there's no provision for custom fields, no plugin/hook surface, and no envelope around the response that future fields could land in cleanly. PATCH semantics aren't spelled out (JSON Merge Patch vs. JSON Patch). Adding any new field will require a v2 release.

### extensibility — operative weakness
Body schema is `{title, body}` with no extension point. The first new requirement (tags, attachments, sharing) will force a v2 cut.

### terseness (8)
Five endpoints, one resource, no ceremony. Error format is minimal and standard. Reads top-to-bottom in under a minute.

## ASI for next round
Add a single extension point — a `metadata: {}` field on the Note body — that lets future additive features ship without a v2 cut. Keep the rest unchanged. This is the highest-leverage move because it lifts the extensibility ceiling at near-zero terseness cost.
