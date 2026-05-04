# Get a tenant sync job by id

**Method:** `GET`
**Path:** `/tenants/{tenantId}/sync-jobs/{id}`
**Resource group:** Sync job (audit)
**Idempotent:** Yes

## Overview

Fetches one `tenant_sync_job` audit row. Useful when polling the outcome of a specific provisioning attempt or projection write.

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `tenantId` | Long | yes | Tenant primary key (must own the job). |
| `id` | Long | yes | Sync-job primary key. |

## Query parameters

_None._

## Request body

_None._

## Response — `200 OK`

Same shape as the entries returned by [list-tenant-sync-jobs.md](list-tenant-sync-jobs.md).

## Errors

| Status | When |
|---|---|
| `404` | No sync-job with that `(tenantId, id)` pair. |

## Notes

- For repeated polling, prefer the list endpoint and look at the most-recent rows.
