# List all tenants belonging to an operator

**Method:** `GET`
**Path:** `/operators/{operatorId}/tenants`
**Resource group:** Tenant
**Idempotent:** Yes

## Overview

Returns all tenants whose `operatorId` matches the path parameter, ordered by id ascending. Includes tenants in every status (`PROVISIONING`, `ACTIVE`, `SUSPENDED`, `DELETED`) — clients filter client-side. Used by admin UIs to render the operator's tenant table.

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `operatorId` | Long | yes | The owning operator's id. |

## Query parameters

_None._

## Request body

_None._

## Response — `200 OK`

```json
[
  {
    "id": 42,
    "operatorId": 1,
    "shortName": "acme",
    "fullName": "Acme Corporation Ltd.",
    "companyName": "Acme Corp",
    "city": "Dhaka",
    "country": "BD",
    "dbHost": "10.10.199.20",
    "dbPort": 3306,
    "dbName": "telcobright_1_acme_42",
    "dbUser": "tenant_acme_42",
    "status": "ACTIVE",
    "createdAt": "2026-05-05T08:14:23.211Z",
    "updatedAt": "2026-05-05T08:14:48.802Z"
  }
]
```

`dbPassRef` is never returned.

## Errors

| Status | When |
|---|---|
| `404` | `operatorId` does not exist. |

## Notes

- For provisioning a new tenant under this operator, see [provision-tenant-under-operator.md](provision-tenant-under-operator.md).
- For mutating an individual tenant, see [update-tenant.md](update-tenant.md).
