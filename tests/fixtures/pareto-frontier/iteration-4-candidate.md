# Notes API v1

## Resources

### Notes
- `GET /v1/notes` — list notes for the authenticated user
- `GET /v1/notes/{id}` — fetch one note
- `POST /v1/notes` — create. Body: `{title, body, metadata: {}}`
- `PATCH /v1/notes/{id}` — update via JSON Merge Patch (RFC 7396)
- `DELETE /v1/notes/{id}` — delete

### Extension points
- `metadata` is an open object on every Note. Servers preserve unknown keys round-trip. Reserved prefix `_` for server-set keys.

### Auth
Bearer token in `Authorization` header. Token issued by existing `/v1/auth/login` endpoint (unchanged).

### Errors
Standard HTTP status codes. Error body: `{error: {code, message}}` (unchanged from v1).

### Versioning
Path prefix (`/v1/`). Additive changes (new fields under `metadata`, new optional body fields) are non-breaking. Breaking changes ship at `/v2/`.
