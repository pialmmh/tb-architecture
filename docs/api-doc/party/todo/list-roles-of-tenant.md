# List all roles defined in a tenant

**Method:** `GET`
**Path:** `/tenants/{tenantId}/roles`
**Resource group:** Auth role
**Idempotent:** Yes

## Overview

Returns every `auth_role` row in the tenant. Roles are tenant-scoped: two tenants can have a role named `admin` independently.

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
    "id": 3,
    "tenantId": 42,
    "name": "tenant-admin",
    "description": "Full access within the tenant",
    "createdAt": "2026-04-23T05:00:00.000Z",
    "updatedAt": "2026-04-23T05:00:00.000Z"
  }
]
```

## Errors

| Status | When |
|---|---|
| `404` | No tenant with that id. |

## Notes

- For role-permission assignment, see [set-permissions-of-role.md](set-permissions-of-role.md).
