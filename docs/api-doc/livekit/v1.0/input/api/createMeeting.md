# createMeeting()

HTTP API call. `POST /api/meetings`. Creates a new meeting row in `SCHEDULED` status; the backing LiveKit room is created lazily on first token issuance, not here.

Authenticated. Roles `AGENT` and `ADMIN` only — `CLIENT` callers are rejected (403). The caller becomes `hostUserId`.

## Params

- `title` (string, required) — display name shown in the room list.
- `scheduledAt` (timestamp, required) — start time; meetings can be joined any time after this once `LIVE`.
- `endTime` (timestamp, optional) — auto-end horizon; the lifecycle pass on listMeetings flips status to `ENDED` after this point.
- `securityMode` (`PUBLIC` | `PRIVATE`, required) — PUBLIC = anyone with the share link; PRIVATE = per-recipient links only.
- `autoAdmit` (bool, optional, default true) — PUBLIC-only knob; when false, joiners enter the waiting room.
- `maxParticipants` (int, optional, default 50) — capacity ceiling enforced at token issuance.
- `recordingEnabled` (bool, optional) — gates the start-recording controls in the UI; recording itself is a host action.
- `muteOnEntry` (bool, optional) — joiners' mics start muted.
- `videoOffOnEntry` (bool, optional) — joiners' cameras start off.
- `participantsCanShareScreen` (bool, optional) — non-host screen-share permission.
- `participantsCanShareFile` (bool, optional) — non-host file-upload permission.
- `eventVisibility` (`PUBLIC` | `HOST_ONLY`, optional) — who can read the meeting-event timeline.

## Computed (not supplied by caller)

- `id` — UUID, used as both DB primary key and the LiveKit room name.
- `roomName` — same value as `id`; we don't separate the two.
- `hostUserId` — taken from the session.

## Fires trigger

- [trigger/meetingProvisioning](../../trigger/meetingProvisioning.md)
