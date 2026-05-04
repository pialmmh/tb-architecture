# Reset an auth user's password

**Method:** `POST`
**Path:** `/tenants/{tenantId}/users/{id}/password`
**Resource group:** Auth user
**Idempotent:** Yes

## Overview

Replaces the user's stored `passwordHash` with a new BCrypt hash. Used by admin-driven password resets. The endpoint does NOT verify the previous password — the auth check is the caller's responsibility.

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `tenantId` | Long | yes | Tenant primary key. |
| `id` | Long | yes | User primary key. |

## Query parameters

_None._

## Request body

```json
{ "password": "N3w-Strong-Password" }
```

| Field | Type | Required | Description |
|---|---|---|---|
| `password` | string | yes | Plaintext; minimum length / complexity enforced. |

## Response — `204 No Content`

Empty body.

## Errors

| Status | When |
|---|---|
| `404` | No user with that `(tenantId, id)` pair. |
| `400` | Password fails policy. |

## Notes

- Active sessions are NOT invalidated; token revocation happens at the auth provider.
