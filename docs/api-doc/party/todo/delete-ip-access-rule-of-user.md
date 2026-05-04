# Delete an IP access rule of an auth user

**Method:** `DELETE`
**Path:** `/tenants/{tenantId}/users/{userId}/ip-rules/{ruleId}`
**Resource group:** Auth user · IP access rules
**Idempotent:** Yes

## Overview

Hard-deletes an `ip_access_rule` row. IP rules don't carry historical-audit value; the row is removed entirely.

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `tenantId` | Long | yes | Tenant primary key. |
| `userId` | Long | yes | User primary key. |
| `ruleId` | Long | yes | Rule primary key. |

## Query parameters

_None._

## Request body

_None._

## Response — `204 No Content`

Empty body.

## Errors

| Status | When |
|---|---|
| `404` | No rule with that `(tenantId, userId, ruleId)` triple. |

## Notes

- A `tenant_sync_job` row is enqueued with `entityType=IP_ACCESS_RULE`, `operation=DELETE`.
