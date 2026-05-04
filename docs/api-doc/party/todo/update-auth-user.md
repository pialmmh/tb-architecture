# Update an auth user (partial)

**Method:** `PATCH`
**Path:** `/tenants/{tenantId}/users/{id}`
**Resource group:** Auth user
**Idempotent:** Yes

## Overview

Partial-update of a tenant user. Only fields present in the request body are written. `id`, `tenantId`, `partnerId`, `passwordHash` are immutable via this endpoint — for password changes use [reset-auth-user-password.md](reset-auth-user-password.md); to move a user between partners, delete and re-create.

A `tenant_sync_job` is enqueued.

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `tenantId` | Long | yes | Tenant primary key. |
| `id` | Long | yes | User primary key. |

## Query parameters

_None._

## Request body

```json
{
  "firstName": "Renamed",
  "lastName": "User",
  "phone": "+88019999999",
  "userStatus": "SUSPENDED"
}
```

| Field | Type | Mutable | Description |
|---|---|---|---|
| `firstName` / `lastName` / `phone` | string | yes | Profile fields. |
| `userStatus` | enum | yes | `ACTIVE` / `SUSPENDED` / `LOCKED` / `DELETED`. |
| `email` | string | yes | Reassign login email; must remain tenant-unique. |
| `pbxUuid` | string | yes | PBX user id (telco-specific). |
| `resellerDbName` | string | yes | Per-reseller DB name override. |
| `id`, `tenantId`, `partnerId`, `passwordHash` | — | **no** | Ignored. |

## Response — `200 OK`

The updated `AuthUser`.

## Errors

| Status | When |
|---|---|
| `404` | No user with that `(tenantId, id)` pair. |
| `400` | Invalid enum / malformed payload. |
| `409` | Email collides with another user in the tenant. |
