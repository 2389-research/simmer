# Notes API v1

## Resources

### Notes
- `GET /v1/notes` — list notes for the authenticated user
- `GET /v1/notes/{id}` — fetch one note
- `POST /v1/notes` — create a note. Body: `{title, body}`
- `PATCH /v1/notes/{id}` — update title or body
- `DELETE /v1/notes/{id}` — delete

### Auth
Bearer token in `Authorization` header. Token issued by existing `/v1/auth/login` endpoint (unchanged from prior version).

### Errors
Standard HTTP status codes. Error body: `{error: {code, message}}`.

### Versioning
Path prefix (`/v1/`). Breaking changes ship under `/v2/`.
