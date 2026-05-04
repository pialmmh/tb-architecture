# Delete a role

**Method:** `DELETE`
**Path:** `/tenants/{tenantId}/roles/{id}`
**Resource group:** Auth role
**Idempotent:** Yes

## Overview

Removes a role. Removes role-permission and user-role associations as well. Soft-vs-hard delete behaviour: the role row is hard-deleted; user-role join rows referencing it are removed in the same operation. Pre-existing `auth_user.roles` cached on JWTs continue to grant access until the token expires.

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `tenantId` | Long | yes | Tenant primary key. |
| `id` | Long | yes | Role primary key. |

## Query parameters

_None._

## Request body

_None._

## Response — `204 No Content`

Empty body.

## Errors

| Status | When |
|---|---|
| `404` | No role with that `(tenantId, id)` pair. |

## Notes

- A `tenant_sync_job` row is enqueued to mirror the deletion in the projection DB.
