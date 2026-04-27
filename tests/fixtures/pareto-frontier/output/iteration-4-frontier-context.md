# Frontier Context for Iteration 4

## Current iteration's failure modes
- **extensibility (6)** — The `metadata` open object is the right minimal extension point, but there's no plugin namespacing pattern (`extensions.{name}` from iter-3 is absent) and no event/webhook surface. Score capped well below iter-3's 9.

## Frontier members and their strengths

### iter-3 — best on: extensibility
**Score profile:** backwards_compat: 5, extensibility: 9, terseness: 4

**Operative passage:**
> - **`metadata`** — arbitrary user-controlled key/value pairs on every Note. Servers preserve unknown keys round-trip. Reserved namespace `_*` for server-set keys.
> - **`extensions`** — namespaced plugin payloads. Each registered plugin owns a top-level key (e.g., `extensions.sharing`, `extensions.tags`, `extensions.encryption`). Plugins register schemas separately at `/v1/plugins/{name}/schema`.

**Why it works (judge evidence):**
> The `extensions.{name}` namespace pattern is the strongest move — it lets multiple plugins coexist without naming collisions and makes plugin presence trivially detectable by clients.

**Known weakness:**
> Surface area roughly tripled. Six new concepts (envelope, metadata, extensions, hooks, vendor MIME, field selection) before a reader gets to the resource itself. Plugin schema registration at `/v1/plugins/{name}/schema` adds a whole second resource family. Reads more like a framework than an API.

---

### iter-4 — best on: backwards_compat, terseness
**Score profile:** backwards_compat: 8, extensibility: 6, terseness: 9

**Operative passage:**
> - `GET /v1/notes` — list notes for the authenticated user
> - `GET /v1/notes/{id}` — fetch one note
> - `POST /v1/notes` — create. Body: `{title, body, metadata: {}}`
> - `PATCH /v1/notes/{id}` — update via JSON Merge Patch (RFC 7396)
> - `DELETE /v1/notes/{id}` — delete

**Why it works (judge evidence):**
> Five endpoints, one resource. One paragraph for the extension point, two lines for auth, one line for errors. Reads in 30 seconds. Slightly tighter than iter-1 because the explicit JSON Merge Patch reference removes a reader's need to ask 'which patch flavor?'.

**Known weakness:**
> The `metadata` open object is exactly the right minimal extension point — future additive features can land there without a v2 cut. Up from iter-1's score of 5 because there's now a place for additions to go. Not as high as iter-3 because there's no plugin namespacing pattern (`extensions.{name}` from iter-3 was the strongest single move and it's not present here) and no event/webhook surface.

---

## Relevance to this iteration

iter-4 is the current best-balanced candidate but its weakest dim is extensibility (6). iter-3 holds best_at for extensibility (9) — its operative passage shows the specific transferable move: a top-level `extensions` object on the Note body where each registered plugin owns a namespaced key (`extensions.sharing`, `extensions.tags`, `extensions.encryption`), with `metadata` reserved for arbitrary user-controlled key/value pairs and `_*` reserved for server-set keys. The technique to apply: add a single `extensions: {}` field next to iter-4's existing `metadata: {}` in the create body, and add one short line stating the namespace rule (one top-level key per plugin under `extensions.{name}`). Constraint: preserve iter-4's terseness (9) and backwards_compat (8) — that means do NOT borrow the rest of iter-3's surface (no JSON:API response envelope, no `/v1/plugins/{name}/schema` resource family, no hooks endpoint, no vendor MIME types, no field-selection or include parameters). Keep the existing five-endpoint shape, the unchanged `/v1/auth/login`, and the unchanged `{error: {code, message}}` envelope. The judgment's own ASI confirms this: borrow the namespace pattern only, leaving iter-3's framework-shaped surface area on the dropped pile. Expected effect: extensibility climbs toward 8 while terseness stays above 8.

## Recently dropped — avoid these failure modes
- iter-1: Closed `{title, body}` body with no extension point — first new feature would force a v2 cut.
- iter-2: Forced token reauth and renamed error envelope, breaking every existing client.
