# listUsers()

HTTP API call. `GET /api/users`. Searches the user directory by name / username / email; used by the participant-picker on the meeting create / edit form.

Authenticated. Roles `AGENT` and `ADMIN` only — `CLIENT` callers are rejected (403). Result capped at 50 rows.

## Params

- `q` (query, optional) — substring to match against name/username/email; empty returns recent users.
- `role` (query, optional) — `Role` enum filter (e.g. `AGENT`).

## Fires trigger

- [trigger/userDirectory](../../trigger/userDirectory.md)
