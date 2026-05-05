# streamEvents()

HTTP API call. `GET /api/meetings/{meetingId}/events/stream`. Server-Sent Events stream. The server sends each new `MeetingEvent` as it is appended; clients reconnect with `EventSource`'s built-in retry, then call [backfillEvents](backfillEvents.md) with the last seen id to catch up on the gap.

permitAll on the route — gated by `Meeting.eventVisibility` like backfill. The `SseEmitter` uses no timeout; reverse proxies must disable response buffering for the stream to flow.

## Params

- `meetingId` (path, required)

## Fires trigger

- [trigger/eventReplay](../../trigger/eventReplay.md)
