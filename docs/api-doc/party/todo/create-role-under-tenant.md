# Create a new role under a tenant

**Method:** `POST`
**Path:** `/tenants/{tenantId}/roles`
**Resource group:** Auth role
**Idempotent:** No (each call creates a new row; `name` must be unique within the tenant).

## Overview

Inserts a new `auth_role` for the tenant. Permissions can be attached afterwards via [set-permissions-of-role.md](set-permissions-of-role.md).

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `tenantId` | Long | yes | Tenant primary key. |

## Query parameters

_None._

## Request body

```json
{
  "name": "tenant-admin",
  "description": "Full access within the tenant"
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string ≤ 80 | yes | Role name; unique within the tenant. |
| `description` | string ≤ 240 | no | Free-form description. |

## Response — `200 OK`

The created `AuthRole`.

## Errors

| Status | When |
|---|---|
| `400` | Missing `name`. |
| `404` | No tenant with that id. |
| `409` | Another role in the tenant has the same `name`. |

## Notes

- Side effect: a `tenant_sync_job` row with `entityType=AUTH_ROLE`, `operation=INSERT`.
