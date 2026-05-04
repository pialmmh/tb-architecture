# Create a new operator user (platform super-admin)

**Method:** `POST`
**Path:** `/operator-users`
**Resource group:** Operator user
**Idempotent:** No (each call creates a new row).

## Overview

Inserts a new `OperatorUser` — a super-admin scoped to one operator. The supplied `password` is BCrypt-hashed (cost 12) before storage. The new user starts in `ACTIVE` status.

## Path parameters

_None._

## Query parameters

_None._

## Request body

```json
{
  "operatorId": 1,
  "email": "ops-lead@telcobright.com",
  "password": "S3cret-At-Least-12-Chars",
  "firstName": "Ops",
  "lastName": "Lead",
  "phone": "+88029999999"
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `operatorId` | Long | yes | Owning operator. |
| `email` | string ≤ 160 | yes | Unique across all operator users. |
| `password` | string | yes | Plain text — BCrypt-hashed server-side. Minimum length and complexity enforced via Bean Validation. |
| `firstName` | string ≤ 80 | no | First name. |
| `lastName` | string ≤ 80 | no | Last name. |
| `phone` | string ≤ 40 | no | Contact phone. |

## Response — `200 OK`

The created `OperatorUser` (without `passwordHash`).

## Errors

| Status | When |
|---|---|
| `400` | Missing required field, weak password, or `operatorId` does not exist. |
| `409` | Another operator-user already has the same `email`. |

## Notes

- See [reset-operator-user-password.md](reset-operator-user-password.md) for password rotation.
- See [set-operator-user-status.md](set-operator-user-status.md) for activate / suspend toggles.
