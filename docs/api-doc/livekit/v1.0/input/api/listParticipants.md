# listParticipants()

HTTP API call. `GET /api/meetings/{meetingId}/participants`. Returns the meeting's roster (the pre-call list of who's expected), not the live in-room participants — the latter come from the LiveKit SDK on the client.

Authenticated. Each row is enriched with the underlying `User` projection if it carries a `userId`; external guests are returned with their email + name only.

## Params

- `meetingId` (path, required) — meeting id.

## Fires trigger

- [trigger/participantRosterManagement](../../trigger/participantRosterManagement.md)
