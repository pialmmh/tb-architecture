# knockStatus()

HTTP API call. `GET /api/join/{token}/knock/{knockId}`. Polled by the waiting-room joiner after [joinMeeting](joinMeeting.md) put them in the queue. Returns one of:

- `PENDING` — host hasn't decided yet.
- `ADMITTED` — body includes the freshly-minted LK token; the claim is recorded here.
- `DENIED` — body includes the host's optional reason.
- `EXPIRED` — the queue entry was reaped by [waitingRoomSweep](../clock/waitingRoomSweep.md).

permitAll. The poll cadence is the client's choice (UI uses ~1s).

## Params

- `token` (path, required)
- `knockId` (path, required) — id returned from joinMeeting.

## Fires trigger

- [trigger/magicLinkClaim](../../trigger/magicLinkClaim.md)
