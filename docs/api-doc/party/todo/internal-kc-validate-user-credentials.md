# Internal Keycloak SPI — validate user credentials

**Method:** `POST`
**Path:** `/internal/kc/users/validate-credentials`
**Resource group:** Internal Keycloak SPI
**Idempotent:** Yes
**Audience:** LAN-only.

## Overview

Verifies a `(realm, username, password)` triple against the BCrypt hash stored in `auth_user.password_hash`. Used by Keycloak's User Storage SPI when the user is authenticated against the realm and Party owns the credentials.

The endpoint is **constant-time** in successful vs. unknown-user paths to avoid user enumeration via timing.

## Path parameters

_None._

## Query parameters

_None._

## Request body

```json
{
  "realm": "tenant-acme",
  "username": "user@bbc.example",
  "password": "user-supplied-plaintext"
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `realm` | string | yes | Keycloak realm name. |
| `username` | string | yes | Typically the email. |
| `password` | string | yes | Plaintext. |

## Response — `200 OK`

```json
{ "valid": true }
```

| Field | Type | Description |
|---|---|---|
| `valid` | bool | `true` if the password matched and the user is `ACTIVE`; `false` otherwise. |

## Errors

| Status | When |
|---|---|
| `400` | Missing field, or realm not mapped to a tenant. |
| `401` | Missing / invalid SPI shared secret. |

## Notes

- Suspended / locked / deleted users return `{"valid": false}` even with the correct password.
- `auth_user.last_login_at` is NOT updated by this endpoint — it is updated only by the user-facing [login-with-email-and-password.md](login-with-email-and-password.md). Login-time updates from Keycloak are forwarded asynchronously by the SPI.
