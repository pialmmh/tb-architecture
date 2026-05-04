# Log in with email and password

**Method:** `POST`
**Path:** `/auth/login`
**Resource group:** Authentication
**Idempotent:** Yes (each call returns a new token, but produces no persistent side-effect beyond updating `lastLoginAt`).

## Overview

Authenticates an `auth_user` (tenant-scoped) by email + password and issues a short-lived JWT. The user's tenant and partner are derived from the row matching the email; multi-tenant disambiguation by email is not currently supported (each email belongs to at most one tenant in the master DB).

This endpoint is the **legacy Party-issued JWT path**. The strategic direction is for Keycloak to issue OIDC tokens via the User Storage SPI (`/internal/kc/*`); this endpoint will remain for service-to-service flows that don't sit behind Keycloak.

## Path parameters

_None._

## Query parameters

_None._

## Request body

```json
{
  "email": "user@acme.example",
  "password": "user-supplied-plaintext"
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `email` | string | yes | The user's email; must match an `auth_user.email` row. |
| `password` | string | yes | Plaintext; verified against the BCrypt `passwordHash` stored in master. |

## Response — `200 OK`

```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiJ9...",
  "refreshToken": "eyJhbGciOiJIUzI1NiJ9...",
  "expiresInSeconds": 900,
  "tenantId": 42,
  "userId": 7
}
```

| Field | Type | Description |
|---|---|---|
| `accessToken` | string | HS256 JWT; lifetime per `party.jwt.access-ttl-minutes` (default 15). |
| `refreshToken` | string | HS256 JWT; lifetime per `party.jwt.refresh-ttl-minutes` (default 1440). |
| `expiresInSeconds` | int | Access-token TTL in seconds. |
| `tenantId` | Long | Tenant the user belongs to. |
| `userId` | Long | The `auth_user.id`. |

## Errors

| Status | When |
|---|---|
| `400` | Missing email or password. |
| `401` | Email not found or password mismatch. The error body does not distinguish — to avoid user enumeration. |
| `403` | User exists but `userStatus` is not `ACTIVE`. |

## Notes

- `lastLoginAt` is updated on success.
- Token signing key comes from `party.jwt.secret` (env-var resolved); claims include `sub`, `tenantId`, `partnerId`, `roles`.
- Brute-force protection is NOT implemented at this layer — handle upstream (rate limit at gateway, account lockout via [set-operator-user-status.md](set-operator-user-status.md) for super-admins).
