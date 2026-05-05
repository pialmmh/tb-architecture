# participantRosterManagement

Manage the meeting's *roster* — the pre-call list of "who's expected". Distinct from the live LK room participant set, which the SDK tracks client-side.

## Activated by

- input: [input/api/listParticipants](../input/api/listParticipants.md)
- input: [input/api/addParticipant](../input/api/addParticipant.md)
- input: [input/api/removeParticipant](../input/api/removeParticipant.md)

## What it does

- **list**: read `meeting_participants` rows for the meeting. Enrich each row carrying a `userId` with the matching `User` projection (name, username) for display. Guests (no `userId`) are returned with their `email` + `name` only.
- **add**: insert one row. The two flavours — known user (`userId`) vs guest (`email`+`name`) — store on the same table; `userId` xor `email` is non-null. Idempotent on the natural key. Rejected (409) if the meeting is in a terminal state.
- **remove**: delete by `MeetingParticipant.id`. Same terminal-state guard.

## Produces

- [output/meetingParticipant](../output/meetingParticipant.md)

## Idempotency & retries

`add` is idempotent on `(meetingId, userId)` and `(meetingId, email)`. `remove` is idempotent (deleting an already-deleted row is a no-op).

## Notes

Roster management does **not** kick live participants out of the LK room — that's the moderation surface (see [moderationAction](./moderationAction.md)). The two are separate by design: removing somebody from the roster after a meeting doesn't disconnect them mid-call.
