# Create a new auth user under a partner

**Method:** `POST`
**Path:** `/tenants/{tenantId}/partners/{partnerId}/users`
**Resource group:** Auth user
**Idempotent:** No (each call creates a new row).

## Overview

Inserts a new `auth_user` belonging to a partner. Password is BCrypt-hashed (cost 12) before storage. Optional `roleIds` can be supplied to assign roles in the same call. A `tenant_sync_job` is enqueued so the projection DB receives the user.

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `tenantId` | Long | yes | Tenant primary key. |
| `partnerId` | Long | yes | Partner primary key. |

## Query parameters

_None._

## Request body

```json
{
  "email": "user@bbc.example",
  "password": "S3cret-At-Least-12-Chars",
  "firstName": "Some",
  "lastName": "User",
  "phone": "+88012345678",
  "roleIds": [3, 5]
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `email` | string ≤ 160 | yes | Unique within the tenant. |
| `password` | string | yes | Plaintext; BCrypt-hashed server-side. |
| `firstName` | string ≤ 80 | no | First name. |
| `lastName` | string ≤ 80 | no | Last name. |
| `phone` | string ≤ 40 | no | Contact phone. |
| `roleIds` | Long[] | no | Role ids to attach immediately. Roles must belong to the same tenant. |

## Response — `200 OK`

The created `AuthUser` (without `passwordHash`). `userStatus` defaults to `ACTIVE`.

## Errors

| Status | When |
|---|---|
| `400` | Missing required field, weak password, or `roleIds` reference roles in a different tenant. |
| `404` | No partner with that `(tenantId, partnerId)` pair. |
| `409` | Another user in the tenant has the same email. |

## Notes

- Role assignment can also be done after creation via [set-auth-user-roles.md](set-auth-user-roles.md).
