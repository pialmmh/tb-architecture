# Update an operator (partial)

**Method:** `PATCH`
**Path:** `/operators/{id}`
**Resource group:** Operator
**Idempotent:** Yes (applying the same patch again is a no-op).

## Overview

Partial-update of an operator's mutable fields. Only the fields present in the request body are written; unset fields are preserved. `shortName` and `id` are immutable — attempts to change them are silently ignored.

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `id` | Long | yes | Operator primary key. |

## Query parameters

_None._

## Request body

```json
{
  "fullName": "Telcobright Pvt Ltd (Bangladesh)",
  "phone": "+88029988877",
  "status": "SUSPENDED"
}
```

| Field | Type | Mutable | Description |
|---|---|---|---|
| `fullName` | string | yes | Human-readable name. |
| `operatorType` | enum | yes | `PROVIDER` / `RESELLER` / `INTEGRATOR`. |
| `companyName` | string | yes | Legal company name. |
| `address1` | string | yes | Address line. |
| `city` | string | yes | City. |
| `country` | string | yes | Country code. |
| `phone` | string | yes | Phone. |
| `email` | string | yes | Email. |
| `status` | string | yes | `ACTIVE` / `SUSPENDED` / `DELETED`. Use [delete-operator.md](delete-operator.md) for the soft-delete shortcut. |
| `shortName` | string | **no** | Ignored. |
| `id` | Long | **no** | Ignored. |

## Response — `200 OK`

The updated `Operator`.

## Errors

| Status | When |
|---|---|
| `404` | No operator with that id. |
| `400` | Invalid enum / malformed payload. |

## Notes

- Setting `status="SUSPENDED"` does **not** cascade to tenants. Suspending a tenant requires a separate call.
