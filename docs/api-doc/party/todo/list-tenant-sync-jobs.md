# List sync jobs for a tenant

**Method:** `GET`
**Path:** `/tenants/{tenantId}/sync-jobs`
**Resource group:** Sync job (audit)
**Idempotent:** Yes

## Overview

Returns the audit trail of provisioning + projection-write jobs for one tenant, newest first. A `tenant_sync_job` row is written for every mutation in master (operator/tenant/partner/user/role/permission CRUD) and for each provisioning step. The row records the workflow id, the activity outcome, attempts, timing, and any error.

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `tenantId` | Long | yes | Tenant primary key. |

## Query parameters

| Name | Type | Required | Default | Description |
|---|---|---|---|---|
| `status` | enum | no | (all) | Filter to one of `PROVISIONING` / `RUNNING` / `SUCCESS` / `FAILED`. |
| `limit` | int | no | `50` | Max rows returned. Server-capped at 500. |

## Request body

_None._

## Response — `200 OK`

```json
[
  {
    "id": 1234,
    "tenantId": 42,
    "workflowId": "tenant-provision-42-1714896823211",
    "runId": "0192a4d8-...",
    "entityType": "TENANT_PROVISION",
    "entityId": "42",
    "operation": "PROVISION",
    "status": "SUCCESS",
    "attempts": 1,
    "startedAt": "2026-05-05T08:14:23.300Z",
    "finishedAt": "2026-05-05T08:14:48.802Z",
    "error": null,
    "createdAt": "2026-05-05T08:14:23.211Z"
  }
]
```

## Errors

| Status | When |
|---|---|
| `404` | No tenant with that id. |

## Notes

- `entityType` values: `TENANT_PROVISION`, `TENANT`, `PARTNER`, `AUTH_USER`, `AUTH_ROLE`, `AUTH_PERMISSION`, `IP_ACCESS_RULE`, `UI_MENU_PERMISSION`.
- `operation` values: `PROVISION`, `INSERT`, `UPDATE`, `DELETE`, `ROLE_PERMISSIONS`, `USER_ROLES`.
