# removeParticipant()

HTTP API call. `DELETE /api/meetings/{meetingId}/participants/{participantId}`. Removes a roster entry. Does **not** kick a live participant from the LiveKit room — for that, see [moderation/kick](kickParticipant.md).

Authenticated. Host or `ADMIN`. Rejected with 409 if the meeting is in a terminal state.

## Params

- `meetingId` (path, required)
- `participantId` (path, required) — `MeetingParticipant.id`.

## Fires trigger

- [trigger/participantRosterManagement](../../trigger/participantRosterManagement.md)
