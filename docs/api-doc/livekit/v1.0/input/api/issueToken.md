# issueToken()

HTTP API call. `POST /api/meetings/{id}/token`. Mints a LiveKit JWT that the browser uses to join the meeting's LiveKit room.

Authenticated session OR a host/admin bypass for invited guests. This endpoint is also where the meeting's lifecycle is "armed" — the first non-ghost join flips status from `SCHEDULED` to `LIVE` and stamps `startedAt`. A `PARTICIPANT_JOINED` event is appended.

**Known authz gap (v1.0):** the participant-roster gate is not enforced — any authenticated caller who knows the meeting id can mint a token. The caller is currently trusted on `name` and `role` for guests too. Tightening this is on the security backlog (`S1`).

## Params

- `id` (path, required) — meeting id.
- `role` (query, optional) — `host` | `participant` | `observer` | `ghost`. `ghost` minted for hidden join paths (e.g. recording bot) — does not flip lifecycle to `LIVE`.
- `name` (query, optional) — display name baked into the JWT identity.

## Computed (not supplied by caller)

- LiveKit room is created on the LK server lazily by the SDK on first publish — we don't pre-create rooms.
- The JWT bakes in screen-share and recording permissions from the meeting's current settings; later setting changes do **not** retroactively rewrite issued tokens.

## Fires trigger

- [trigger/tokenIssuance](../../trigger/tokenIssuance.md)
