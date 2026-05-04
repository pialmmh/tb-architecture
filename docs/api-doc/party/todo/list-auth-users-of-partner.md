# List auth users belonging to one partner

**Method:** `GET`
**Path:** `/tenants/{tenantId}/partners/{partnerId}/users`
**Resource group:** Auth user
**Idempotent:** Yes

## Overview

Returns the `auth_user` rows that belong to one partner. Used in partner-detail UIs to render the user table.

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `tenantId` | Long | yes | Tenant primary key. |
| `partnerId` | Long | yes | Partner primary key. |

## Query parameters

_None._

## Request body

_None._

## Response — `200 OK`

Same row shape as [list-all-auth-users-of-tenant.md](list-all-auth-users-of-tenant.md), filtered to the supplied partner.

## Errors

| Status | When |
|---|---|
| `404` | No partner with that `(tenantId, partnerId)` pair. |
