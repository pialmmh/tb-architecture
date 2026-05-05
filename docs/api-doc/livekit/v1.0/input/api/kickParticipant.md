# kickParticipant()

HTTP API call. `POST /api/meetings/{meetingId}/moderate/{identity}/kick`. Removes a participant from the LK room. Their existing JWT is invalidated by the server-side eviction; re-issuing requires going through [issueToken](issueToken.md) or a new magic link claim. Appends `PARTICIPANT_KICKED`.

Authenticated. Host or `ADMIN`.

## Params

- `meetingId` (path, required)
- `identity` (path, required)

## Fires trigger

- [trigger/moderationAction](../../trigger/moderationAction.md)
