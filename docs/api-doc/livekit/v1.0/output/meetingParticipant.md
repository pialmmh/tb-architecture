name: meetingParticipant
type: json + DB row (`meeting_participants`)
trigger: [participantRosterManagement](../trigger/participantRosterManagement.md)

A roster entry — pre-call "who's expected", not a live LK room participant.

- `id` — primary key
- `meetingId`
- `userId` (nullable) — set for known users
- `email`, `name` (nullable) — set for external guests; mutually exclusive with `userId`
- `role` — `HOST` | `COHOST` | `PARTICIPANT` | `OBSERVER`

Natural key for idempotent upsert is `(meetingId, userId)` for known users and `(meetingId, email)` for guests. The list endpoint enriches `userId` rows with the matching `User` projection at read time.
