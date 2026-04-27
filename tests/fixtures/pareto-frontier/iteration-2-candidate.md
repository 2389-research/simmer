# Notes API v1.1

## Resources

### Notes
- `GET /v1.1/notes` — list notes
- `GET /v1.1/notes/{id}` — fetch one
- `POST /v1.1/notes` — create. Body: `{title, body, created_at, updated_at}`
- `POST /v1.1/notes/{id}/update` — update (RPC-style, replaces PATCH from v1)
- `POST /v1.1/notes/{id}/delete` — delete (RPC-style)
- `GET /v1.1/notes/search?q=...` — search

### Auth
New token format. Old `/v1/auth/login` tokens are NOT accepted — clients must re-authenticate via `/v1.1/auth/login`.

### Errors
`{error_code, error_message, error_details}` (renamed from v1's `{error: {code, message}}`).

### Versioning
Path prefix bumped to `/v1.1/`. v1 endpoints will be removed in 30 days.
