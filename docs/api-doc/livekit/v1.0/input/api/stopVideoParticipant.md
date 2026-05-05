# stopVideoParticipant()

HTTP API call. `POST /api/meetings/{meetingId}/moderate/{identity}/stop-video`. Mutes every video track currently published by a participant. Equivalent of "Stop video" in the moderation menu. The participant must re-enable their camera themselves.

Authenticated. Host or `ADMIN`.

## Params

- `meetingId` (path, required)
- `identity` (path, required)

## Fires trigger

- [trigger/moderationAction](../../trigger/moderationAction.md)
