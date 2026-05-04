# List all operator users

**Method:** `GET`
**Path:** `/operator-users`
**Resource group:** Operator user
**Idempotent:** Yes

## Overview

Returns every `OperatorUser` row across all operators. Operator users are platform super-admins, scoped to a single operator (`operatorId`); they sit ABOVE tenant-scoped `auth_user` rows in the privilege graph and exist only in the master DB. Used by the platform admin UI to render the user table.

## Path parameters

_None._

## Query parameters

_None._

## Request body

_None._

## Response — `200 OK`

```json
[
  {
    "id": 1,
    "operatorId": 1,
    "email": "admin@telcobright.com",
    "firstName": "Sys",
    "lastName": "Admin",
    "phone": "+88029999999",
    "status": "ACTIVE",
    "lastLoginAt": "2026-05-04T22:01:14.123Z",
    "createdAt": "2026-04-22T19:21:00.000Z",
    "updatedAt": "2026-05-04T22:01:14.123Z"
  }
]
```

`passwordHash` is `@JsonIgnore` and never returned.

## Errors

| Status | When |
|---|---|
| `500` | Database unreachable. |

## Notes

- See [create-operator-user.md](create-operator-user.md) and [reset-operator-user-password.md](reset-operator-user-password.md) for mutations.
