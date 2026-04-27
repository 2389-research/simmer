# Iteration 3 Judgment

## Scores

- backwards_compat: 5
- extensibility: 9
- terseness: 4

Composite: 6.0/10

## Per-criterion evidence

### backwards_compat (5)
Auth is unchanged — good. Endpoint paths unchanged from v1 — good. But the response envelope shift from `{title, body}`-shaped Notes to JSON:API `{data: [...], links, meta}` silently breaks any consumer parsing the v1 shape. Error envelope shift (v1's `{error: {code, message}}` → v1's-new `{errors: [...]}`) is the same problem. The intent is "extensions are additive" but the envelope itself is a breaking move.

### extensibility (9)
This is the operative section: the resource has FIVE distinct extension surfaces — `metadata` (open key/value), `extensions` (namespaced plugin payloads), webhook event-type registration, vendor MIME types for custom serialization, and field/include selection. Plugins can ship without core changes. The `extensions.{name}` namespace pattern is the strongest move — it lets multiple plugins coexist without naming collisions and makes plugin presence trivially detectable by clients. Hooks endpoint with custom event-type registration extends the surface to async too.

### terseness (4)
Surface area roughly tripled. Six new concepts (envelope, metadata, extensions, hooks, vendor MIME, field selection) before a reader gets to the resource itself. Plugin schema registration at `/v1/plugins/{name}/schema` adds a whole second resource family. Reads more like a framework than an API.

## ASI for next round
Extensibility is great but too much surface lands at once. Pick the highest-leverage extension point — `extensions.{name}` namespacing with the response envelope — and drop hooks, vendor MIME, and field selection. Those can ship as additive plugins later under the namespace pattern this round establishes.
