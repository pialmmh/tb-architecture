# meetingProvisioning

Stand up a new meeting row. There is no LiveKit-side provisioning step — LK rooms are created lazily on first publish, so this trigger is purely a DB insert.

## Activated by

- input: [input/api/createMeeting](../input/api/createMeeting.md)

## What it does

1. Generate UUID `id` (also used as the LK `roomName`).
2. Bind `hostUserId` to the calling user.
3. Insert the `meetings` row in status `SCHEDULED` with the supplied settings + sensible defaults (`maxParticipants=50`, `eventVisibility=PUBLIC`, etc.).
4. Return the row to the caller.

## Produces

- [output/meeting](../output/meeting.md)

## Idempotency & retries

Not idempotent — every call inserts a new row. Callers wanting deduplication (e.g. "create-or-get for this calendar event") must layer it themselves; the API has no upsert.

## Notes

`CLIENT`-role users are rejected with 403 in the controller layer before this trigger runs. The check is intentionally there and not in `MeetingService.create()` — keeping the service callable from internal automation that bypasses the role check.
