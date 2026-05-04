# Update a partner (partial)

**Method:** `PATCH`
**Path:** `/tenants/{tenantId}/partners/{id}`
**Resource group:** Partner
**Idempotent:** Yes

## Overview

Partial-update of a partner's mutable fields. Only fields present in the request body are written; unset fields are preserved. Mutating a partner enqueues a `tenant_sync_job` so the projection DB stays in sync.

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `tenantId` | Long | yes | Owning tenant. |
| `id` | Long | yes | Partner primary key. |

## Query parameters

_None._

## Request body

```json
{
  "partnerName": "Big Buyer (renamed)",
  "email": "new-billing@bbc.example",
  "status": "SUSPENDED"
}
```

| Field | Mutable | Notes |
|---|---|---|
| All fields documented in [create-partner-under-tenant.md](create-partner-under-tenant.md) | yes | |
| `id`, `tenantId` | **no** | Ignored. |
| `status` | yes | `ACTIVE` / `SUSPENDED` / `DELETED`. |

## Response — `200 OK`

The updated `Partner`.

## Errors

| Status | When |
|---|---|
| `404` | No partner with that `(tenantId, id)` pair. |
| `400` | Invalid enum / malformed payload. |

## Notes

- Side effect: a `tenant_sync_job` row with `entityType=PARTNER`, `operation=UPDATE`.
