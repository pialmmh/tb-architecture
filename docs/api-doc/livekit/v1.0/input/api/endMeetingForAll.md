# endMeetingForAll()

HTTP API call. `POST /api/meetings/{meetingId}/moderate/end`. Hard-ends the meeting: kicks every participant from the LK room, flips `Meeting.status` to `ENDED`, stamps `actualEndedAt`, appends `MEETING_ENDED`, and clears the in-memory event ring buffer for the meeting.

Authenticated. Host or `ADMIN`. Different from "I left the meeting" (which just closes one client) — this terminates the room.

## Params

- `meetingId` (path, required)

## Fires trigger

- [trigger/moderationAction](../../trigger/moderationAction.md)
