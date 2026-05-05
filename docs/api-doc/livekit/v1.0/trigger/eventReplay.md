# eventReplay

Read-side of the event timeline: backfill since a cursor, and live SSE subscription. Both gated by `Meeting.eventVisibility`.

## Activated by

- input: [input/api/backfillEvents](../input/api/backfillEvents.md)
- input: [input/api/streamEvents](../input/api/streamEvents.md)

## What it does

- **backfill**: read events from the per-meeting ring buffer with `id > since`. If `since=0`, returns everything currently buffered. Returns up to `RING_CAP` entries.
- **stream**: register an `SseEmitter` with `MeetingEventService` for the meeting. On every subsequent append (see [eventEmission](./eventEmission.md)), the server pushes the event JSON down each emitter. The emitter has no timeout; clients reconnect via `EventSource`'s built-in retry.

Authorisation for both: if `Meeting.eventVisibility = HOST_ONLY`, the caller must have a session whose user is host or `ADMIN`. `PUBLIC` allows anyone (including unauthenticated guests).

## Produces

- [output/meetingEvent](../output/meetingEvent.md) (the same artifact as eventEmission produces, just replayed)

## Idempotency & retries

Both are pure reads. Resume after a drop: client passes the last seen `id` to backfill, then re-opens the SSE stream — no events are lost as long as the gap stays inside `RING_CAP`.

## Notes

Reverse proxies in front of this service must disable response buffering for `/events/stream` or the SSE messages will batch up and arrive late. Browsers also cap concurrent EventSource connections per origin (~6 in Chrome) — the UI keeps exactly one stream open per meeting and shares it across components.
