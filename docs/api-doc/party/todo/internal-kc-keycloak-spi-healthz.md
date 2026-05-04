# Internal Keycloak SPI — health check

**Method:** `GET`
**Path:** `/internal/kc/healthz`
**Resource group:** Internal Keycloak SPI
**Idempotent:** Yes
**Audience:** LAN-only.

## Overview

Lightweight reachability check for the SPI surface, distinct from the public `/q/health` endpoint. Returns `200 OK` if Party is up AND the master DB is reachable. Used by Keycloak's `UserStorageProvider.test()` callback.

## Path parameters

_None._

## Query parameters

_None._

## Request body

_None._

## Response — `200 OK`

```json
{ "status": "UP" }
```

## Errors

| Status | When |
|---|---|
| `503` | Master DB unreachable. |
| `401` | Missing / invalid SPI shared secret. |
