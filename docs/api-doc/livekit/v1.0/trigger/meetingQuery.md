# meetingQuery

Read-side of the meeting CRUD: list and get-by-id. Listing also opportunistically advances meeting lifecycle, so it isn't pure read.

## Activated by

- input: [input/api/listMeetings](../input/api/listMeetings.md)
- input: [input/api/getMeeting](../input/api/getMeeting.md)

## What it does

- **listMeetings**: filters rows by caller's visibility (`ADMIN` sees all; others see hosted + roster-listed). For each row, runs the auto-end pass: if `endTime` has elapsed and status isn't terminal, flips to `ENDED` and stamps `actualEndedAt`. The auto-end is in-line, not async — we don't run a dedicated cron.
- **getMeeting**: simple find-by-id, no auto-end pass (single-row reads don't need to be the lifecycle driver).

## Produces

- [output/meeting](../output/meeting.md)
- chained: [trigger/eventEmission](./eventEmission.md) — when the auto-end pass actually flips a row, a `MEETING_ENDED` event is appended

## Idempotency & retries

The auto-end side-effect is idempotent (terminal rows are skipped). Repeated calls converge.

## Notes

The "auto-end on every list" pattern is deliberate — it keeps the lifecycle correct without standing up a Quartz scheduler, at the cost of meetings whose `endTime` has passed not auto-ending until *somebody* lists meetings. Acceptable for our scale; would be wrong for a high-volume calendaring system.
