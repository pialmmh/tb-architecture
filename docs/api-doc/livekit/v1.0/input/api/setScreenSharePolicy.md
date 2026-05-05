# setScreenSharePolicy()

HTTP API call. `POST /api/meetings/{meetingId}/moderate/screen-share`. Updates the meeting's `participantsCanShareScreen` flag. Does **not** revoke screen-share permissions on already-issued tokens (those are JWT-baked); only future joins pick up the new value. The UI compensates by listening on a peer-broadcast `meeting-policy` topic so live participants react to the change.

Authenticated. Host or `ADMIN`.

## Params

- `meetingId` (path, required)
- `enabled` (bool, required) — new value for `participantsCanShareScreen`.

## Fires trigger

- [trigger/moderationAction](../../trigger/moderationAction.md)
