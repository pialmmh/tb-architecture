# getUser()

HTTP API call. `GET /api/users/{id}`. Fetches a single user by ID. Returned as a minimal projection (no password hash, no internal fields).

Authenticated. Roles `AGENT` and `ADMIN` only.

## Params

- `id` (path, required) — numeric user id.

## Fires trigger

- [trigger/userDirectory](../../trigger/userDirectory.md)
