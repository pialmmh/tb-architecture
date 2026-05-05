# promoteParticipant()

HTTP API call. `POST /api/meetings/{meetingId}/moderate/{identity}/promote`. Elevates a participant to co-host: flips `isHost` in their LK metadata and grants the full permission set (publish, subscribe, data, etc.) via the LK update-participant API.

Authenticated. Host or `ADMIN`. Effect is live for the current LK session — re-joining picks up host treatment from the metadata flag.

## Params

- `meetingId` (path, required)
- `identity` (path, required)

## Fires trigger

- [trigger/moderationAction](../../trigger/moderationAction.md)
