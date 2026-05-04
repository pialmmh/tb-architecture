# List IP access rules for an auth user

**Method:** `GET`
**Path:** `/tenants/{tenantId}/users/{userId}/ip-rules`
**Resource group:** Auth user · IP access rules
**Idempotent:** Yes

## Overview

Returns the per-user `ip_access_rule` rows. Each rule maps an IP / CIDR to either `ALLOW` or `DENY`. Rules are evaluated at login / API-gateway level (default-deny: a user with no `ALLOW` rule is allowed from anywhere; once any `ALLOW` exists, only listed IPs pass).

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
    "id": 17,
    "tenantId": 42,
    "userId": 11,
    "ip": "203.0.113.0/24",
    "permissionType": "ALLOW",
    "createdAt": "2026-04-25T10:00:00.000Z"
  }
]
```

## Errors

| Status | When |
|---|---|
| `404` | No user with that `(tenantId, userId)` pair. |

## Notes

- `ip` is stored as a 45-char string and may be a single address (`203.0.113.4`) or CIDR (`203.0.113.0/24`).
- `permissionType` values: `ALLOW`, `DENY`.
