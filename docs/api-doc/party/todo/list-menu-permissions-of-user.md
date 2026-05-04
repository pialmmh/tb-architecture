# List menu permissions for an auth user

**Method:** `GET`
**Path:** `/tenants/{tenantId}/users/{userId}/menu-permissions`
**Resource group:** Auth user · UI menu permissions
**Idempotent:** Yes

## Overview

Returns the per-user `ui_menu_permission` rows. Each row maps a UI menu key (e.g. `partners.list`, `billing.invoices`) to a permission level (`NONE`, `READONLY`, `FULL`). The UI consumes this list at login to render or hide menu items and to disable mutation actions.

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `tenantId` | Long | yes | Tenant primary key. |
| `userId` | Long | yes | User primary key. |

## Query parameters

_None._

## Request body

_None._

## Response — `200 OK`

```json
[
  {
    "id": 91,
    "tenantId": 42,
    "userId": 11,
    "menuKey": "partners.list",
    "permissionLevel": "READONLY",
    "createdAt": "2026-04-25T10:30:00.000Z",
    "updatedAt": "2026-04-25T10:30:00.000Z"
  }
]
```

## Errors

| Status | When |
|---|---|
| `404` | No user with that `(tenantId, userId)` pair. |

## Notes

- `permissionLevel` values: `NONE`, `READONLY`, `FULL`.
- The set of valid `menuKey` values is owned by the UI; Party stores them as opaque strings.
