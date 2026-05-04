# Get an operator by id

**Method:** `GET`
**Path:** `/operators/{id}`
**Resource group:** Operator
**Idempotent:** Yes

## Overview

Fetches a single `Operator` row by primary key. Used by admin UIs to render the operator detail page and by internal services that need to resolve an operator's `shortName`, `status`, or contact info.

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `id` | Long | yes | Operator primary key. |

## Query parameters

_None._

## Request body

_None._

## Response — `200 OK`

```json
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
```

## Errors

| Status | When |
|---|---|
| `404` | No operator with that id. |

## Notes

- See [list-all-operators.md](list-all-operators.md) for bulk reads.
