# List all operators

**Method:** `GET`
**Path:** `/operators`
**Resource group:** Operator
**Idempotent:** Yes

## Overview

Returns every `Operator` in the master DB, ordered by id ascending. Operators are the top of the Party graph — each represents one telco operator (e.g. BTCL) and owns many tenants beneath it. This list is small in practice (single-digit rows) so no pagination is provided.

## Path parameters

_None._

## Query parameters

_None._

## Request body

_None._

## Response — `200 OK`

```json
[
  {
    "id": 1,
    "shortName": "telcobright",
    "fullName": "Telcobright Pvt Ltd",
    "operatorType": "PROVIDER",
    "companyName": "Telcobright Pvt Ltd",
    "address1": "Banani, Dhaka",
    "city": "Dhaka",
    "country": "BD",
    "phone": "+88029999999",
    "email": "ops@telcobright.com",
    "status": "ACTIVE",
    "createdAt": "2026-04-22T19:21:00.000Z",
    "updatedAt": "2026-04-22T19:21:00.000Z"
  }
]
```

| Field | Type | Description |
|---|---|---|
| `id` | Long | Operator primary key. |
| `shortName` | string | Slug; unique. |
| `fullName` | string | Human-readable name. |
| `operatorType` | enum | `PROVIDER` / `RESELLER` / `INTEGRATOR`. |
| `status` | string | `ACTIVE` / `SUSPENDED` / `DELETED`. |

## Errors

| Status | When |
|---|---|
| `500` | Database unreachable. |

## Notes

- See [get-operator-by-id.md](get-operator-by-id.md) for single-row reads.
- See [create-operator.md](create-operator.md) for adding a new operator.
