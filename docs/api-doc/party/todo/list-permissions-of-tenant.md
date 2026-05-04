# List all permissions defined in a tenant

**Method:** `GET`
**Path:** `/tenants/{tenantId}/permissions`
**Resource group:** Auth permission
**Idempotent:** Yes

## Overview

Returns every `auth_permission` row in the tenant. Permissions are tenant-scoped fine-grained capabilities (e.g. `partners.read`, `billing.invoice.create`) that get attached to roles via [set-permissions-of-role.md](set-permissions-of-role.md).

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `tenantId` | Long | yes | Tenant primary key. |

## Query parameters

_None._

## Request body

_None._

## Response — `200 OK`

```json
[
  {
    "id": 1,
    "tenantId": 42,
    "name": "partners.read",
    "description": "View partners",
    "createdAt": "2026-04-23T05:00:00.000Z"
  }
]
```

## Errors

| Status | When |
|---|---|
| `404` | No tenant with that id. |
