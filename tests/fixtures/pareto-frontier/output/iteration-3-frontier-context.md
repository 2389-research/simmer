# Frontier Context for Iteration 3

## Current iteration's failure modes
- **terseness (4)** — Surface area roughly tripled. Six new concepts (envelope, metadata, extensions, hooks, vendor MIME, field selection) before a reader gets to the resource itself. Reads more like a framework than an API.
- **backwards_compat (5)** — Response envelope shift to JSON:API `{data, links, meta}` silently breaks consumers parsing the v1 shape; error envelope shift has the same problem.

## Frontier members and their strengths

### iter-1 — best on: backwards_compat, terseness
**Score profile:** backwards_compat: 7, extensibility: 5, terseness: 8

**Operative passage:**
> - `GET /v1/notes` — list notes for the authenticated user
> - `GET /v1/notes/{id}` — fetch one note
> - `POST /v1/notes` — create a note. Body: `{title, body}`
> - `PATCH /v1/notes/{id}` — update title or body
> - `DELETE /v1/notes/{id}` — delete

**Why it works (judge evidence):**
> Five endpoints, one resource, no ceremony. Error format is minimal and standard. Reads top-to-bottom in under a minute.

**Known weakness:**
> Body schema is `{title, body}` with no extension point. The first new requirement (tags, attachments, sharing) will force a v2 cut.

---

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

## Relevance to this iteration

iter-3 scored 4 on terseness — the current iteration's worst dimension — and 5 on backwards_compat. iter-1 holds best_at for both: its operative passage (the five-endpoint list with `{title, body}` create body and standard `{error: {code, message}}`) is the form that produced terseness=8 and backwards_compat=7. The transferable technique for the next generator: roll back the framework-shaped surface (drop hooks, vendor MIME, field selection, the JSON:API response/error envelopes, and the `/v1/plugins/{name}/schema` second resource family) to iter-1's five-endpoint shape, but keep ONE move from iter-3's operative passage — the `extensions.{name}` namespace pattern (e.g., `extensions.sharing`, `extensions.tags`) as a top-level key on the Note body alongside `metadata`. That preserves the strongest extensibility move (the namespace lets plugins coexist without collisions) while restoring iter-1's response/error wire shape so existing v1 consumers don't silently break. Constraint: do NOT reintroduce the JSON:API envelope, vendor MIME types, or hook registration this round — those are what cost terseness.

## Recently dropped — avoid these failure modes
- iter-2: Forced token reauth and renamed error envelope, breaking every existing client.
