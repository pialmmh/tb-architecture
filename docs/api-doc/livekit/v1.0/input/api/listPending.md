# listPending()

HTTP API call. `GET /api/meetings/{meetingId}/pending`. Returns the list of joiners currently waiting for admission, oldest knock first. Drives the host-side WaitingTab in the meeting drawer.

Authenticated. Host or `ADMIN`. Only `PENDING` entries are returned — `ADMITTED` / `DENIED` / `EXPIRED` entries linger briefly in memory for the joiner's poll loop, then are evicted by [waitingRoomSweep](../clock/waitingRoomSweep.md).

## Params

- `meetingId` (path, required)

## Fires trigger

- [trigger/waitingRoomManagement](../../trigger/waitingRoomManagement.md)
