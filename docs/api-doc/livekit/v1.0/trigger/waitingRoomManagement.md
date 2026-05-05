# waitingRoomManagement

The waiting-room queue. Stores pending joiners in process memory (not DB), drains them via host admit/deny, and reaps stale entries on a clock.

## Activated by

- input: [input/api/listPending](../input/api/listPending.md)
- input: [input/api/admitKnock](../input/api/admitKnock.md)
- input: [input/api/denyKnock](../input/api/denyKnock.md)
- clock: [input/clock/waitingRoomSweep](../input/clock/waitingRoomSweep.md)
- chained from: [trigger/magicLinkClaim](./magicLinkClaim.md) — knock-path branch

## What it does

- **list**: return PENDING entries for the meeting, oldest first. Other states are not returned to hosts (they exist only briefly for the joiner's poll loop).
- **admit**: flip the entry's state to `ADMITTED`. Chain into [tokenIssuance](./tokenIssuance.md) to mint the LK token server-side; store it on the `PendingJoiner` so the joiner picks it up via [knockStatus](../input/api/knockStatus.md). Append `PARTICIPANT_JOINED`.
- **deny**: flip to `DENIED` with the host's optional reason. Append `KNOCK_DENIED`.
- **sweep** (clock): mark `PENDING` entries older than 10 min as `TIMED_OUT`; evict resolved entries (`ADMITTED` / `DENIED` / `TIMED_OUT`) older than 10 min from memory.
- **enroll** (chained from claim): construct a `PendingJoiner` and add it to the queue.

## Produces

- [output/pendingJoiner](../output/pendingJoiner.md)
- [output/livekitAccessToken](../output/livekitAccessToken.md) (on admit)
- chained: [trigger/eventEmission](./eventEmission.md) — `PARTICIPANT_JOINED` on admit, `KNOCK_DENIED` on deny

## Idempotency & retries

`admit` is idempotent — repeated admits return the same already-stored token. `deny` is idempotent. The sweep is internally idempotent (already-resolved entries are skipped).

## Notes

The whole queue lives in `WaitingRoomService`'s in-memory map. **Restarting the backend drops every pending knock** — surviving joiners get a `PENDING → not found` transition on their next poll. Acceptable because knocks are short-lived (TTL 10 min) and our deployment is single-instance; would not be acceptable in a HA setup.
