# admitKnock()

HTTP API call. `POST /api/meetings/{meetingId}/pending/{knockId}/admit`. Admits a waiting-room joiner. Mints the LK token server-side and stores it on the in-memory `PendingJoiner` so the joiner picks it up on their next [knockStatus](knockStatus.md) poll. Appends `PARTICIPANT_JOINED`.

Authenticated. Host or `ADMIN`. Idempotent — repeated admits on the same knockId return the same token.

## Params

- `meetingId` (path, required)
- `knockId` (path, required)

## Fires trigger

- [trigger/waitingRoomManagement](../../trigger/waitingRoomManagement.md)
