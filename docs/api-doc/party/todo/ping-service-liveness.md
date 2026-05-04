# Ping the Party service for liveness

**Method:** `GET`
**Path:** `/ping`
**Resource group:** Service health
**Idempotent:** Yes

## Overview

Cheapest possible reachability probe. Returns a fixed string with no DB or downstream calls. Use `/q/health` (Quarkus SmallRye Health) for full readiness — `/ping` is only for "is the JVM up and serving HTTP?".

## Path parameters

_None._

## Query parameters

_None._

## Request body

_None._

## Response — `200 OK`

```
pong
```

Content-Type: `text/plain`.

## Errors

_None — if Party is up the call always succeeds._

## Notes

- For deeper checks: `GET /q/health`, `GET /q/health/live`, `GET /q/health/ready`, `GET /q/metrics`.
