# Add an IP access rule to an auth user

**Method:** `POST`
**Path:** `/tenants/{tenantId}/users/{userId}/ip-rules`
**Resource group:** Auth user · IP access rules
**Idempotent:** No (each call inserts a new row; duplicate `(ip, permissionType)` pairs are allowed).

## Overview

Inserts an IP allow/deny rule for the user. Rules take effect at the next login or token validation; existing sessions are not re-checked.

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
  "ip": "203.0.113.0/24",
  "permissionType": "ALLOW"
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `ip` | string ≤ 45 | yes | IPv4/IPv6 address or CIDR. |
| `permissionType` | enum | yes | `ALLOW` / `DENY`. |

## Response — `200 OK`

The created `IpAccessRule` row.

## Errors

| Status | When |
|---|---|
| `404` | No user with that `(tenantId, userId)` pair. |
| `400` | Malformed `ip`. |

## Notes

- A `tenant_sync_job` row is enqueued with `entityType=IP_ACCESS_RULE`, `operation=INSERT`.
- To remove a rule, use [delete-ip-access-rule-of-user.md](delete-ip-access-rule-of-user.md).
