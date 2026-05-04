# Set an operator user's status

**Method:** `POST`
**Path:** `/operator-users/{id}/status`
**Resource group:** Operator user
**Idempotent:** Yes

## Overview

Sets the `status` of an operator user. Used to suspend a compromised account or to re-activate one. The status is enforced at login time (`POST /auth/login` rejects non-`ACTIVE` users).

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `id` | Long | yes | Operator-user primary key. |

## Query parameters

_None._

## Request body

```json
{ "status": "SUSPENDED" }
```

| Field | Type | Required | Description |
|---|---|---|---|
| `status` | enum | yes | `ACTIVE` / `SUSPENDED` / `LOCKED` / `DELETED`. |

## Response — `204 No Content`

Empty body.

## Errors

| Status | When |
|---|---|
| `404` | No operator-user with that id. |
| `400` | Invalid `status` value. |

## Notes

- Suspending does NOT delete; the row stays. Use [delete-operator-user.md](delete-operator-user.md) for hard removal.
