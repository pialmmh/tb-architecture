# Delete a permission

**Method:** `DELETE`
**Path:** `/tenants/{tenantId}/permissions/{id}`
**Resource group:** Auth permission
**Idempotent:** Yes

## Overview

Hard-deletes an `auth_permission` row. Removes role-permission join rows as well; users whose JWTs already carry the permission claim retain access until token expiry.

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `tenantId` | Long | yes | Tenant primary key. |
| `id` | Long | yes | Permission primary key. |

## Query parameters

_None._

## Request body

_None._

## Response — `204 No Content`

Empty body.

## Errors

| Status | When |
|---|---|
| `404` | No permission with that `(tenantId, id)` pair. |

## Notes

- A `tenant_sync_job` row is enqueued with `entityType=AUTH_PERMISSION`, `operation=DELETE`.
