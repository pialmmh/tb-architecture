# Update a role (partial)

**Method:** `PATCH`
**Path:** `/tenants/{tenantId}/roles/{id}`
**Resource group:** Auth role
**Idempotent:** Yes

## Overview

Partial-update of a role's mutable fields (`name`, `description`). `id` and `tenantId` are immutable.

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `tenantId` | Long | yes | Tenant primary key. |
| `id` | Long | yes | Role primary key. |

## Query parameters

_None._

## Request body

```json
{
  "name": "tenant-admin-renamed",
  "description": "Updated description"
}
```

| Field | Type | Mutable | Description |
|---|---|---|---|
| `name` | string ≤ 80 | yes | Must remain unique within the tenant. |
| `description` | string ≤ 240 | yes | Free-form. |
| `id`, `tenantId` | — | **no** | Ignored. |

## Response — `200 OK`

The updated `AuthRole`.

## Errors

| Status | When |
|---|---|
| `404` | No role with that `(tenantId, id)` pair. |
| `409` | New `name` collides with another role. |
