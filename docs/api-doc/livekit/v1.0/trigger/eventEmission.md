# eventEmission

Append a `MeetingEvent` to the per-meeting in-memory ring buffer and fan out to every active SSE subscriber. Triggered both directly by the client (whitelisted types) and chained from many server-side triggers as a side-effect.

## Activated by

- input: [input/api/emitClientEvent](../input/api/emitClientEvent.md)
- chained from: [trigger/meetingMutation](./meetingMutation.md) — `MEETING_ENDED`, `LOCKED`, `UNLOCKED`
- chained from: [trigger/meetingQuery](./meetingQuery.md) — `MEETING_ENDED` from auto-end
- chained from: [trigger/tokenIssuance](./tokenIssuance.md) — `PARTICIPANT_JOINED`, `MEETING_STARTED`
- chained from: [trigger/magicLinkClaim](./magicLinkClaim.md) — `PARTICIPANT_JOINED` (direct path)
- chained from: [trigger/waitingRoomManagement](./waitingRoomManagement.md) — `PARTICIPANT_JOINED` on admit, `KNOCK_DENIED` on deny
- chained from: [trigger/recordingControl](./recordingControl.md) — `RECORDING_STARTED`, `RECORDING_STOPPED`
- chained from: [trigger/moderationAction](./moderationAction.md) — `PARTICIPANT_KICKED`, `PARTICIPANT_BLOCKED`, `MEETING_ENDED`

## What it does

1. For client emits: validate `type` against `CLIENT_EMITTABLE` whitelist (`HAND_RAISED`, `HAND_LOWERED`, `SCREEN_SHARE_STARTED`, `SCREEN_SHARE_STOPPED`, `PARTICIPANT_LEFT`, `FILE_SHARED`, `MENTION`, `POLL_CREATED`, `POLL_CLOSED`). Reject anything else with 400.
2. Append the event to `MeetingEventService`'s per-meeting bounded ring buffer. Assigns a monotonic `id`.
3. Push the event to every registered `SseEmitter` for the meeting. Failures on individual emitters drop them silently from the listener set.

## Produces

- [output/meetingEvent](../output/meetingEvent.md)

## Idempotency & retries

Not idempotent — each call appends a new entry. Clients dedupe by `id` if needed (the SSE + backfill flow is built around this).

## Notes

The ring buffer is **in-process and bounded** (`MeetingEventService.RING_CAP`). After `endMeetingForAll`, the buffer is cleared. Backend restart drops all timelines. This is acceptable for our usage (timelines are an in-call affordance, not an audit log) but is the wrong choice if you need durable event history — orchestrix-v2 should swap to a DB-backed event log.

Server-only event types (e.g. `MEETING_STARTED`, `RECORDING_STARTED`, `PARTICIPANT_KICKED`) cannot be spoofed by clients because they aren't on the whitelist.
