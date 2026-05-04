# Delete a partner (soft delete)

**Method:** `DELETE`
**Path:** `/tenants/{tenantId}/partners/{id}`
**Resource group:** Partner
**Idempotent:** Yes

## Overview

Marks a partner as `DELETED`. Soft delete only. The row is retained for audit and to keep `auth_user.partnerId` foreign keys valid. A `tenant_sync_job` is enqueued to mirror the change in the projection DB.

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `tenantId` | Long | yes | Owning tenant. |
| `id` | Long | yes | Partner primary key. |

## Query parameters

_None._

## Request body

_None._

## Response — `204 No Content`

Empty body.

## Errors

| Status | When |
|---|---|
| `404` | No partner with that `(tenantId, id)` pair. |

## Notes

- Re-deleting an already-deleted partner returns `204`.
- To re-activate, use [update-partner.md](update-partner.md) with `{"status":"ACTIVE"}`.
