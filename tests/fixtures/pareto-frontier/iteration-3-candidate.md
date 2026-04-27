# Notes API — Extensible Edition

## Resources

### Notes
- `GET /v1/notes` — list. Response: `{data: [...], links: {...}, meta: {...}}` (JSON:API style envelope)
- `GET /v1/notes/{id}` — fetch one
- `POST /v1/notes` — create. Body: `{title, body, metadata: {}, extensions: {}}`
- `PATCH /v1/notes/{id}` — update via JSON Merge Patch (RFC 7396)
- `DELETE /v1/notes/{id}` — delete

### Extension points

- **`metadata`** — arbitrary user-controlled key/value pairs on every Note. Servers preserve unknown keys round-trip. Reserved namespace `_*` for server-set keys.
- **`extensions`** — namespaced plugin payloads. Each registered plugin owns a top-level key (e.g., `extensions.sharing`, `extensions.tags`, `extensions.encryption`). Plugins register schemas separately at `/v1/plugins/{name}/schema`.
- **Hooks** — `POST /v1/hooks` registers webhooks for `note.created`, `note.updated`, `note.deleted`. Custom event types can be registered via `POST /v1/hooks/event-types`.
- **Custom MIME types** — `Accept: application/vnd.notes.{plugin}+json` routes responses through plugin-specific serializers.
- **Field selection** — `?fields=title,metadata.priority` lets clients project arbitrary subtrees.
- **Embedded resources** — `?include=author,comments,attachments` hydrates related resources inline.

### Auth
Bearer token, unchanged from v1. Plugin tokens supported via `Authorization: Plugin <name> <token>` for plugin-scoped operations.

### Errors
JSON:API error objects: `{errors: [{status, code, title, detail, source, meta}]}`. Plugins may add namespaced fields under `meta`.

### Versioning
Path prefix `/v1/`. Field additions are non-breaking (envelopes, metadata, extensions absorb them). Breaking changes only at major version bumps.
