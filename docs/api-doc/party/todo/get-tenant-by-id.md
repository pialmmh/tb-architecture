# Get a tenant by id

**Method:** `GET`
**Path:** `/tenants/{id}`
**Resource group:** Tenant
**Idempotent:** Yes

## Overview

Fetches a single `Tenant` row by primary key. Used by admin UIs to render the tenant detail page and by services that need the projection DB connection coordinates (`dbHost`, `dbPort`, `dbName`, `dbUser`).

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `id` | Long | yes | Tenant primary key. |

## Query parameters

_None._

## Request body

_None._

## Response — `200 OK`

Same shape as the entries returned by [list-tenants-of-operator.md](list-tenants-of-operator.md). `dbPassRef` is never returned.

## Errors

| Status | When |
|---|---|
| `404` | No tenant with that id. |

## Notes

- The connection password is fetched at runtime via the `dbPassRef` env-var name (e.g. `PARTY_TENANT_DB_PASS`); it is not part of the API response.
