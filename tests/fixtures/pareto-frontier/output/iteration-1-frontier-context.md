# Frontier Context for Iteration 1

## Current iteration's failure modes
- **extensibility (5)** — Body schema is `{title, body}` with no extension point. The first new requirement (tags, attachments, sharing) will force a v2 cut.
- **backwards_compat (7)** — No explicit deprecation header pattern documented, so future deprecations will be ad hoc.

## Frontier members and their strengths

### iter-1 — best on: backwards_compat, terseness, extensibility
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

## Relevance to this iteration

iter-1 is the only frontier member, so no cross-member comparison is possible yet. Its strengths are terseness (8: five endpoints, one resource, no ceremony) and backwards_compat (7: reuses existing `/v1/auth/login` endpoint and conventional path-prefix versioning). Its operative weakness is extensibility (5): the Note body `{title, body}` has no extension point, so the first new requirement (tags, attachments, sharing) will force a v2 cut. The next iteration should add a single extension surface — e.g., a `metadata: {}` field — without disturbing the five-endpoint shape that produced the terseness win.

## Recently dropped — avoid these failure modes
(none yet)
