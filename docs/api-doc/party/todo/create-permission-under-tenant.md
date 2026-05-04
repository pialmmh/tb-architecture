# Create a new permission under a tenant

**Method:** `POST`
**Path:** `/tenants/{tenantId}/permissions`
**Resource group:** Auth permission
**Idempotent:** No (each call creates a new row).

## Overview

Inserts a new `auth_permission` for the tenant. Permission `name` should be unique within the tenant; the API does not enforce uniqueness today (will be added with V3 migration). Attach to roles via [set-permissions-of-role.md](set-permissions-of-role.md).

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `tenantId` | Long | yes | Tenant primary key. |

## Query parameters

_None._

## Request body

```json
{
  "name": "billing.invoice.create",
  "description": "Create an invoice"
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string ≤ 120 | yes | Permission identifier. Convention: `<domain>.<entity>.<action>` lowercase. |
| `description` | string ≤ 240 | no | Free-form. |

## Response — `200 OK`

The created `AuthPermission`.

## Errors

| Status | When |
|---|---|
| `400` | Missing `name`. |
| `404` | No tenant with that id. |

## Notes

- Side effect: a `tenant_sync_job` row with `entityType=AUTH_PERMISSION`, `operation=INSERT`.
