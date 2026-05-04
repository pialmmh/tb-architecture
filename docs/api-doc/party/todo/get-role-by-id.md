# Get a role by id

**Method:** `GET`
**Path:** `/tenants/{tenantId}/roles/{id}`
**Resource group:** Auth role
**Idempotent:** Yes

## Overview

Fetches one `auth_role` row scoped to a tenant.

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `tenantId` | Long | yes | Tenant primary key. |
| `id` | Long | yes | Role primary key. |

## Query parameters

_None._

## Request body

_None._

## Response — `200 OK`

Same shape as the entries returned by [list-roles-of-tenant.md](list-roles-of-tenant.md).

## Errors

| Status | When |
|---|---|
| `404` | No role with that `(tenantId, id)` pair. |
