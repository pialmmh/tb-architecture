# Update a tenant (partial)

**Method:** `PATCH`
**Path:** `/tenants/{id}`
**Resource group:** Tenant
**Idempotent:** Yes

## Overview

Partial-update of a tenant's mutable, non-infrastructure fields. Only fields present in the request body are written; unset fields are preserved. The DB coordinates (`dbHost`, `dbPort`, `dbName`, `dbUser`, `dbPassRef`), `id`, `operatorId`, and `shortName` are immutable and silently ignored if supplied.

Mutating a tenant **also enqueues a `tenant_sync_job`** so the per-tenant projection DB stays in sync with master.

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `id` | Long | yes | Tenant primary key. |

## Query parameters

_None._

## Request body

```json
{
  "fullName": "Acme Corp (Asia)",
  "companyName": "Acme Asia Ltd.",
  "phone": "+88029999999",
  "email": "asia@acme.example",
  "status": "SUSPENDED"
}
```

| Field | Type | Mutable | Description |
|---|---|---|---|
| `fullName` | string | yes | Human-readable name. |
| `companyName` | string | yes | Legal company name. |
| `address1`, `city`, `country`, `phone`, `email` | string | yes | Contact info. |
| `status` | string | yes | `ACTIVE` / `SUSPENDED` / `DELETED`. |
| `id`, `operatorId`, `shortName`, `dbHost`, `dbPort`, `dbName`, `dbUser`, `dbPassRef` | — | **no** | Ignored. |

## Response — `200 OK`

The updated `Tenant`.

## Errors

| Status | When |
|---|---|
| `404` | No tenant with that id. |
| `400` | Invalid status / malformed payload. |

## Notes

- Side effect: a `tenant_sync_job` row with `entityType=TENANT`, `operation=UPDATE` is written and dispatched. Watch [list-tenant-sync-jobs.md](list-tenant-sync-jobs.md) for the projection-write status.
- To rename the slug (`shortName`) you must delete and re-provision — DB names are derived from it.
