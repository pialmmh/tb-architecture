# Reset an operator user's password

**Method:** `POST`
**Path:** `/operator-users/{id}/password`
**Resource group:** Operator user
**Idempotent:** Yes (calling twice with the same payload is equivalent — final hash is the same).

## Overview

Replaces the operator user's stored `passwordHash` with a new BCrypt hash of the supplied plaintext. Used by admin password-reset flows. Does NOT verify the old password — call sites must enforce the appropriate auth check before invoking.

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `id` | Long | yes | Operator-user primary key. |

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
| `404` | No operator-user with that id. |
| `400` | Password fails policy. |

## Notes

- Active sessions for this user are NOT invalidated server-side. Token revocation is the auth provider's responsibility.
