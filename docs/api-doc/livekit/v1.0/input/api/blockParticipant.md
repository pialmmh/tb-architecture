# blockParticipant()

HTTP API call. `POST /api/meetings/{meetingId}/moderate/{identity}/block`. Two side-effects: (a) inserts a `BlockedUser` row keyed on whatever identifier the caller knows about the offender, and (b) immediately kicks them from the room. Future `joinMeeting` calls matching the same key are rejected. Appends `PARTICIPANT_BLOCKED`.

Authenticated. Host or `ADMIN`. Caller passes the identifying metadata; at least one of `userId`, `contactChannel`, `deviceFingerprint` is required.

## Params

- `meetingId` (path, required)
- `identity` (path, required) — used for the kick step.
- `metadata` (object, required) — any of `{userId, contactChannel, deviceFingerprint}`.
- `reason` (string, optional) — recorded on the BlockedUser row and surfaced in the timeline.

## Fires trigger

- [trigger/moderationAction](../../trigger/moderationAction.md)
