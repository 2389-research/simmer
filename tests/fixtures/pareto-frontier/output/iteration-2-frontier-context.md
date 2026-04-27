# Frontier Context for Iteration 2

## Current iteration's failure modes
- **backwards_compat (3)** — Token reauth requirement breaks every existing client on day one. Path bump to `/v1.1/` plus 30-day removal of `/v1/` is aggressive deprecation with no migration path. Error envelope rename silently breaks consumers parsing the old shape.
- **extensibility (4)** — Switching from REST PATCH to RPC-style `/update` and `/delete` is more verbose without buying any extension surface. Still no `metadata` field, still no envelope.
- **terseness (6)** — Endpoint count went from 5 to 6, RPC paths are longer, error format gained a third field.

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

iter-2 was strictly dominated by iter-1 on every criterion (bc 3<7, ext 4<5, ter 6<8) and dropped. The frontier still holds only iter-1, whose extensibility score of 5 remains the standing weakness. The next iteration should roll back to iter-1's five-endpoint shape and add a single extension point (a `metadata: {}` field on the Note body) — that lifts extensibility without touching the auth surface, error envelope, or path versioning that produced iter-1's backwards_compat and terseness wins. Avoid iter-2's moves: do not bump the path prefix, do not rename the error envelope, do not switch to RPC-style endpoints.

## Recently dropped — avoid these failure modes
- iter-2: Forced token reauth and renamed error envelope, breaking every existing client.
