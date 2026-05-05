# demoteParticipant()

HTTP API call. `POST /api/meetings/{meetingId}/moderate/{identity}/demote`. Inverse of [promoteParticipant](promoteParticipant.md): clears `isHost` in the participant's LK metadata and resets their permissions to participant-default.

Authenticated. Host or `ADMIN`.

## Params

- `meetingId` (path, required)
- `identity` (path, required)

## Fires trigger

- [trigger/moderationAction](../../trigger/moderationAction.md)
