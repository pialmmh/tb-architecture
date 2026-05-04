# Internal Keycloak SPI — search users

**Method:** `GET`
**Path:** `/internal/kc/users/search`
**Resource group:** Internal Keycloak SPI
**Idempotent:** Yes
**Audience:** LAN-only.

## Overview

Free-text search across `auth_user` rows in one realm/tenant. Used by the Keycloak admin console "Users" page when Party is the user-storage backend. Matches against `email`, `firstName`, `lastName` (case-insensitive substring).

## Path parameters

_None._

## Query parameters

| Name | Type | Required | Default | Description |
|---|---|---|---|---|
| `realm` | string | yes | — | Keycloak realm name. |
| `q` | string | no | — | Search string; if absent, returns the first page of all users. |
| `first` | int | no | `0` | Offset for pagination. |
| `max` | int | no | `20` | Page size. Server-capped at 200. |

## Request body

_None._

## Response — `200 OK`

```json
[
  {
    "id": "11",
    "username": "user@bbc.example",
    "email": "user@bbc.example",
    "firstName": "Some",
    "lastName": "User",
    "enabled": true,
    "emailVerified": true,
    "tenantId": 42,
    "partnerId": 7,
    "roles": ["tenant-user"]
  }
]
```

## Errors

| Status | When |
|---|---|
| `400` | Realm not mapped to a tenant. |
| `401` | Missing / invalid SPI shared secret. |
