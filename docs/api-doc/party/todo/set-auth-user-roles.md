# Set the roles assigned to an auth user

**Method:** `POST`
**Path:** `/tenants/{tenantId}/users/{id}/roles`
**Resource group:** Auth user
**Idempotent:** Yes (the supplied list fully replaces the previous assignment).

## Overview

Replaces the user's role assignment with the supplied list of role ids. To add a single role, fetch the current set first, append, and POST the merged list. All roles must belong to the same tenant.

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `tenantId` | Long | yes | Tenant primary key. |
| `id` | Long | yes | User primary key. |

## Query parameters

_None._

## Request body

```json
{ "roleIds": [3, 5, 9] }
```

| Field | Type | Required | Description |
|---|---|---|---|
| `roleIds` | Long[] | yes | Role ids to assign. Empty array removes all roles. |

## Response — `204 No Content`

Empty body.

## Errors

| Status | When |
|---|---|
| `404` | No user with that `(tenantId, id)` pair. |
| `400` | One or more `roleIds` reference roles in a different tenant. |

## Notes

- Side effect: a `tenant_sync_job` row with `entityType=AUTH_USER`, `operation=USER_ROLES`.
