# muteParticipant()

HTTP API call. `POST /api/meetings/{meetingId}/moderate/{identity}/mute`. Mutes one specific track or all audio tracks of a participant via the LK server-side API.

Authenticated. Host or `ADMIN`. Acts directly on the LK room — the participant cannot un-mute server-muted tracks themselves until they actively re-publish.

## Params

- `meetingId` (path, required)
- `identity` (path, required) — LiveKit participant identity.
- `trackSid` (string, optional) — if present, mute only that track; otherwise scan and mute every audio track on the participant.
- `muted` (bool, optional, default true) — pass false to unmute (rarely used; participants normally unmute themselves).

## Fires trigger

- [trigger/moderationAction](../../trigger/moderationAction.md)
