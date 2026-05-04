# Get an auth user by id

**Method:** `GET`
**Path:** `/tenants/{tenantId}/users/{id}`
**Resource group:** Auth user
**Idempotent:** Yes

## Overview

Fetches one `auth_user` row by primary key, scoped to a tenant.

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `tenantId` | Long | yes | Tenant primary key. |
| `id` | Long | yes | User primary key. |

## Query parameters

_None._

## Request body

_None._

## Response — `200 OK`

Same shape as the entries returned by [list-all-auth-users-of-tenant.md](list-all-auth-users-of-tenant.md).

## Errors

| Status | When |
|---|---|
| `404` | No user with that `(tenantId, id)` pair. |
