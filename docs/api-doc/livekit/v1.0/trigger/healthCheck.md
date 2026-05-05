# healthCheck

Trivial liveness signal. Exists as its own trigger because the call is an external contract (orchestrators check it) even though there's no business meaning behind it.

## Activated by

- input: [input/api/health](../input/api/health.md)

## What it does

Returns `{ok: true}` synchronously. No DB read, no downstream call.

## Produces

- [output/healthStatus](../output/healthStatus.md)

## Idempotency & retries

Pure read; safe to call any number of times. No state.

## Notes

If we ever add real readiness checks (DB ping, LK reachability), this is where they go — the controller stays trivial.
