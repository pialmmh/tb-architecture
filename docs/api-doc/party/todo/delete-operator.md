# Delete an operator (soft delete)

**Method:** `DELETE`
**Path:** `/operators/{id}`
**Resource group:** Operator
**Idempotent:** Yes.

## Overview

Marks an operator as `DELETED` in master. This is a **soft delete** — the row stays in `party_master.operator` so foreign keys from tenants and audit jobs remain valid. The operator is hidden from default list responses going forward.

This call does **not** cascade-delete tenants, projection databases, or Keycloak realms. Cleanup of those artifacts is a separate ops task.

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `id` | Long | yes | Operator primary key. |

## Query parameters

_None._

## Request body

_None._

## Response — `204 No Content`

Empty body.

## Errors

| Status | When |
|---|---|
| `404` | No operator with that id. |

## Notes

- Re-deleting an already-deleted operator returns `204` (idempotent).
- To re-activate, use [update-operator.md](update-operator.md) with `{"status":"ACTIVE"}`.
