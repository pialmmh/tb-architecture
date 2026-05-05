# getMeeting()

HTTP API call. `GET /api/meetings/{id}`. Fetches one meeting by id.

permitAll — the meeting id is unguessable (UUID) so this is the read path used by the join-by-link UI before the user has a session. Non-existence returns 404.

## Params

- `id` (path, required) — meeting UUID.

## Fires trigger

- [trigger/meetingQuery](../../trigger/meetingQuery.md)
