# Delete a tenant (soft delete)

**Method:** `DELETE`
**Path:** `/tenants/{id}`
**Resource group:** Tenant
**Idempotent:** Yes

## Overview

Marks a tenant as `DELETED` in master. **Soft delete only** — the row, the per-tenant projection DB, and the per-tenant Keycloak realm are NOT physically dropped. This keeps audit trails and partner / user references valid. Physical cleanup is a separate ops task (out of scope for this API).

A `tenant_sync_job` row is enqueued to mark the projection DB's mirror tenant row as `DELETED`.

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `id` | Long | yes | Tenant primary key. |

## Query parameters

_None._

## Request body

_None._

## Response — `204 No Content`

Empty body.

## Errors

| Status | When |
|---|---|
| `404` | No tenant with that id. |

## Notes

- Re-deleting an already-deleted tenant returns `204` (idempotent).
- To re-activate, use [update-tenant.md](update-tenant.md) with `{"status":"ACTIVE"}` — but if the projection DB or realm has been physically dropped, manual re-provisioning is required.
