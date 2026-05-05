# addParticipant()

HTTP API call. `POST /api/meetings/{meetingId}/participants`. Adds a known user (by id) or an external guest (by email + name) to the meeting's roster. Idempotent on `(meetingId, userId)` and `(meetingId, email)`.

Authenticated. Host or `ADMIN`. Rejected with 409 if the meeting is in a terminal state (`ENDED`, `CANCELLED`).

## Params

Either:

- `userId` (long, required) — id of an existing `User`.

Or:

- `email` (string, required) — guest email.
- `name` (string, optional) — guest display name.

Plus:

- `role` (`HOST` | `COHOST` | `PARTICIPANT` | `OBSERVER`, optional, default `PARTICIPANT`).

## Fires trigger

- [trigger/participantRosterManagement](../../trigger/participantRosterManagement.md)
