# Set the permissions assigned to a role

**Method:** `POST`
**Path:** `/tenants/{tenantId}/roles/{id}/permissions`
**Resource group:** Auth role
**Idempotent:** Yes (the supplied list fully replaces the previous assignment).

## Overview

Replaces the role's permission set with the supplied list of permission ids. All permissions must belong to the same tenant.

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `tenantId` | Long | yes | Tenant primary key. |
| `id` | Long | yes | Role primary key. |

## Query parameters

_None._

## Request body

```json
{ "permissionIds": [1, 2, 7] }
```

| Field | Type | Required | Description |
|---|---|---|---|
| `permissionIds` | Long[] | yes | Permission ids to assign. Empty array clears all. |

## Response — `204 No Content`

Empty body.

## Errors

| Status | When |
|---|---|
| `404` | No role with that `(tenantId, id)` pair. |
| `400` | One or more `permissionIds` reference permissions in a different tenant. |

## Notes

- Side effect: a `tenant_sync_job` row with `entityType=AUTH_ROLE`, `operation=ROLE_PERMISSIONS`.
