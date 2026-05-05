# health()

HTTP API call. `GET /api/health`. Liveness probe — used by load balancers, container orchestrators, and external monitoring. Always returns `{ok: true}`; the call itself succeeding is the signal.

permitAll. No body.

## Params

(none)

## Fires trigger

- [trigger/healthCheck](../../trigger/healthCheck.md)
