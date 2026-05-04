# Get a partner by id

**Method:** `GET`
**Path:** `/tenants/{tenantId}/partners/{id}`
**Resource group:** Partner
**Idempotent:** Yes

## Overview

Fetches one `Partner` row scoped to a tenant. Used by admin UIs to render the partner detail page.

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `tenantId` | Long | yes | Owning tenant. |
| `id` | Long | yes | Partner primary key. |

## Query parameters

_None._

## Request body

_None._

## Response — `200 OK`

Same shape as the entries returned by [list-partners-of-tenant.md](list-partners-of-tenant.md).

## Errors

| Status | When |
|---|---|
| `404` | No partner with that `(tenantId, id)` pair. |

## Notes

- Use [get-partner-extra-info.md](get-partner-extra-info.md) to fetch the optional extended row (NID, trade license, addresses 2–4, …).
