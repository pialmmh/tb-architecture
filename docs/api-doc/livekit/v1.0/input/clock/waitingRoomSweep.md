# waitingRoomSweep (clock)

Scheduled task. Fixed delay of 60 s, runs in `WaitingRoomService.sweep()`. Two responsibilities:

1. Mark `PENDING` knock entries older than 10 minutes as `TIMED_OUT` so polling joiners get a definitive answer instead of hanging.
2. Forget resolved entries (`ADMITTED`, `DENIED`, `TIMED_OUT`) older than 10 minutes so the in-memory map doesn't grow unboundedly.

## Cadence

`@Scheduled(fixedDelay = 60000)` — every 60 seconds, single-threaded. Delay is from end-of-previous-run to start-of-next, so pauses during long sweeps don't pile up.

## Fires trigger

- [trigger/waitingRoomManagement](../../trigger/waitingRoomManagement.md)
