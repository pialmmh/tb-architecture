# Delete an operator user (soft delete)

**Method:** `DELETE`
**Path:** `/operator-users/{id}`
**Resource group:** Operator user
**Idempotent:** Yes

## Overview

Marks an operator user as `DELETED`. Soft delete only — row is retained for audit. To suspend rather than delete, use [set-operator-user-status.md](set-operator-user-status.md).

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `id` | Long | yes | Operator-user primary key. |

## Query parameters

_None._

## Request body

_None._

## Response — `204 No Content`

Empty body.

## Errors

| Status | When |
|---|---|
| `404` | No operator-user with that id. |

## Notes

- Re-deleting an already-deleted user returns `204`.
