# muteAll()

HTTP API call. `POST /api/meetings/{meetingId}/moderate/mute-all`. Iterates every participant in the LK room and mutes their audio tracks, **except** those carrying `isHost` or `isAdmin` in their metadata.

Authenticated. Host or `ADMIN`. Best-effort: failures on individual participants are swallowed and the loop continues.

## Params

- `meetingId` (path, required)

## Fires trigger

- [trigger/moderationAction](../../trigger/moderationAction.md)
