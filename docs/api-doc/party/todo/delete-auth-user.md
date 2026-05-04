# Delete an auth user (soft delete)

**Method:** `DELETE`
**Path:** `/tenants/{tenantId}/users/{id}`
**Resource group:** Auth user
**Idempotent:** Yes

## Overview

Marks an auth user as `DELETED`. Soft delete only; row is retained for audit and to keep references valid. A `tenant_sync_job` is enqueued.

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `tenantId` | Long | yes | Tenant primary key. |
| `id` | Long | yes | User primary key. |

## Query parameters

_None._

## Request body

_None._

## Response — `204 No Content`

Empty body.

## Errors

| Status | When |
|---|---|
| `404` | No user with that `(tenantId, id)` pair. |

## Notes

- Re-deleting an already-deleted user returns `204`.
- To re-activate, use [update-auth-user.md](update-auth-user.md) with `{"userStatus":"ACTIVE"}`.
