# backfillEvents()

HTTP API call. `GET /api/meetings/{meetingId}/events`. Returns the meeting's event timeline since a cursor id. Used by clients on tab-mount to fill in everything that happened before they subscribed to the [SSE stream](streamEvents.md).

permitAll on the route — gated in the controller by `Meeting.eventVisibility`: `PUBLIC` allows anyone; `HOST_ONLY` requires a session whose user is host or `ADMIN`. Records older than the in-memory ring buffer's capacity are not retrievable.

## Params

- `meetingId` (path, required)
- `since` (query, optional, default 0) — return events with `id > since`. First call passes 0; resume after a drop passes the last seen id.

## Fires trigger

- [trigger/eventReplay](../../trigger/eventReplay.md)
