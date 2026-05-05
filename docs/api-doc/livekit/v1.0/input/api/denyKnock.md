# denyKnock()

HTTP API call. `POST /api/meetings/{meetingId}/pending/{knockId}/deny`. Denies a waiting-room joiner with an optional reason that is surfaced to them on their next poll. Appends `KNOCK_DENIED` to the timeline.

Authenticated. Host or `ADMIN`.

## Params

- `meetingId` (path, required)
- `knockId` (path, required)
- `reason` (string, optional) — human-readable; shown to the denied joiner.

## Fires trigger

- [trigger/waitingRoomManagement](../../trigger/waitingRoomManagement.md)
