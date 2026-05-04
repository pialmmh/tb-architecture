# Upsert a menu permission for an auth user

**Method:** `PUT`
**Path:** `/tenants/{tenantId}/users/{userId}/menu-permissions`
**Resource group:** Auth user · UI menu permissions
**Idempotent:** Yes

## Overview

Creates or updates a single `(menuKey, permissionLevel)` mapping for a user. PUT semantics — for each `menuKey` there is at most one row, replaced on each call.

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `tenantId` | Long | yes | Tenant primary key. |
| `userId` | Long | yes | User primary key. |

## Query parameters

_None._

## Request body

```json
{
  "menuKey": "partners.list",
  "permissionLevel": "FULL"
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `menuKey` | string ≤ 100 | yes | UI menu identifier. |
| `permissionLevel` | enum | yes | `NONE` / `READONLY` / `FULL`. |

## Response — `200 OK`

The full `UiMenuPermission` row.

## Errors

| Status | When |
|---|---|
| `404` | No user with that `(tenantId, userId)` pair. |
| `400` | Invalid `permissionLevel`. |

## Notes

- A `tenant_sync_job` row is enqueued with `entityType=UI_MENU_PERMISSION`, `operation=UPSERT`.
- To remove an entry, set `permissionLevel=NONE`.
