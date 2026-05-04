# Internal Keycloak SPI — find a user by username

**Method:** `GET`
**Path:** `/internal/kc/users/by-username`
**Resource group:** Internal Keycloak SPI
**Idempotent:** Yes
**Audience:** LAN-only — consumed by the Party User Storage SPI running inside Keycloak. **Never expose publicly.**

## Overview

Resolves an `auth_user` by `(realm, username)` and returns it in the Keycloak SPI shape (`KcUserView`). Keycloak realms are mapped 1:1 to Party tenants (`realm = "tenant-" + tenant.shortName`).

## Path parameters

_None._

## Query parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `realm` | string | yes | Keycloak realm name; resolved to a tenant id internally. |
| `username` | string | yes | Username — typically the user's email. |

## Request body

_None._

## Response — `200 OK`

```json
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
```

## Errors

| Status | When |
|---|---|
| `404` | No user matches `(realm, username)`. |
| `400` | Realm not mapped to a tenant. |
| `401` | Missing or invalid `PARTY_KC_INTEGRATION_SECRET`. |

## Notes

- Authentication: shared-secret header `X-Party-Kc-Token` matched against `party.kc-integration.secret`.
- Reachability: only over the WireGuard overlay or local LAN.
