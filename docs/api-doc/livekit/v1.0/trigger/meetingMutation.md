# meetingMutation

Mutate an existing meeting: settings updates, status transitions, lock toggle. Drives a fair amount of timeline activity as a side-effect.

## Activated by

- input: [input/api/updateMeeting](../input/api/updateMeeting.md)
- input: [input/api/setMeetingLock](../input/api/setMeetingLock.md)

## What it does

1. Load the `meetings` row; 404 if missing.
2. Authorise: caller must be `hostUserId` or `ADMIN`. Else 403.
3. For status transitions, validate against `MeetingService.isAllowedTransition(from, to)`. Illegal transitions return 409 with body `{"error": "meeting is X — ..."}`.
4. Apply the supplied fields. If status flipped to `LIVE` and `startedAt` is null, stamp it. If status flipped to terminal (`ENDED` / `CANCELLED`), stamp `actualEndedAt`.
5. Persist.
6. Append the matching event to the timeline: `MEETING_ENDED` on terminal flip, `LOCKED` / `UNLOCKED` on lock change.

## Produces

- [output/meeting](../output/meeting.md)
- chained: [trigger/eventEmission](./eventEmission.md) — for lifecycle and lock events

## Idempotency & retries

Status transitions are guarded by the legal-transition check, so retrying a "happens to already be ENDED" call returns 409 rather than re-stamping or re-emitting. Field updates with the same values are no-ops.

## Notes

The mutation set is broad and intentionally not split per-field — the contract is "PUT replaces the supplied subset of fields", not REST-purist per-field PATCH endpoints.
