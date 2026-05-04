# Create a new operator

**Method:** `POST`
**Path:** `/operators`
**Resource group:** Operator
**Idempotent:** No (each call creates a new row; `shortName` must be unique).

## Overview

Inserts a new `Operator` row. Operators are created infrequently (once per onboarded telco). The new operator starts in `ACTIVE` status; tenants can be provisioned under it immediately via [provision-tenant-under-operator.md](provision-tenant-under-operator.md).

## Path parameters

_None._

## Query parameters

_None._

## Request body

```json
{
  "shortName": "telcobright",
  "fullName": "Telcobright Pvt Ltd",
  "operatorType": "PROVIDER",
  "companyName": "Telcobright Pvt Ltd",
  "address1": "Banani, Dhaka",
  "city": "Dhaka",
  "country": "BD",
  "phone": "+88029999999",
  "email": "ops@telcobright.com"
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `shortName` | string ≤ 40 | yes | Lowercase slug; unique across all operators. Used to derive per-tenant DB names downstream. |
| `fullName` | string ≤ 200 | yes | Human-readable name. |
| `operatorType` | enum | yes | `PROVIDER` / `RESELLER` / `INTEGRATOR`. |
| `companyName` | string ≤ 200 | no | Legal company name. |
| `address1` | string ≤ 200 | no | First address line. |
| `city` | string ≤ 80 | no | City. |
| `country` | string ≤ 80 | no | Country code or name. |
| `phone` | string ≤ 40 | no | Contact phone. |
| `email` | string ≤ 120 | no | Contact email. |

`id`, `status` (= `ACTIVE`), `createdAt`, `updatedAt` are server-populated.

## Response — `200 OK`

The created `Operator` (same shape as [get-operator-by-id.md](get-operator-by-id.md)).

## Errors

| Status | When |
|---|---|
| `400` | Missing required field, malformed `shortName`, or invalid `operatorType`. |
| `409` | Another operator already has the same `shortName`. |

## Notes

- Creating an operator does **not** create any tenants. Provision tenants separately.
- Re-running with a different `shortName` always creates a new row — there is no idempotency key.
