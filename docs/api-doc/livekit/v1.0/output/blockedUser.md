name: blockedUser
type: DB row (`blocked_users`)
trigger: [moderationAction](../trigger/moderationAction.md) (block input)

Records that a particular identifier is barred from joining a particular meeting. Three identifier columns; at least one is non-null per row:

- `userId` — for known logged-in users
- `contactChannel` — for guests joining via PRIVATE links (the `invitedEmail`)
- `deviceFingerprint` — for guests joining via PUBLIC links (the `meet_dev` cookie value)

Plus:

- `meetingId` — block scope (per-meeting, not per-host or global)
- `reason` (nullable) — surfaced in the timeline event
- `blockedBy`, `blockedAt`

Checked at the start of `joinMeeting`; matching joiners get 403. Schema is current as of migration V9 (the V8/V9 reshape splits identifiers into discrete nullable columns rather than one polymorphic column).

**Caveat:** the device-fingerprint check is bypassable by clearing cookies. The userId and contactChannel components remain in force for those identifier types.
