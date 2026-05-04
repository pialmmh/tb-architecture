# Internal Keycloak SPI — find a user by id

**Method:** `GET`
**Path:** `/internal/kc/users/by-id`
**Resource group:** Internal Keycloak SPI
**Idempotent:** Yes
**Audience:** LAN-only.

## Overview

Resolves an `auth_user` by `(realm, id)` (Party's `auth_user.id`) and returns the Keycloak SPI shape. Used by Keycloak's User Storage SPI to refresh user attributes on token validation.

## Path parameters

_None._

## Query parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `realm` | string | yes | Keycloak realm name. |
| `id` | string | yes | Party user id (numeric, sent as string per the SPI contract). |

## Request body

_None._

## Response — `200 OK`

Same shape as [internal-kc-find-user-by-username.md](internal-kc-find-user-by-username.md).

## Errors

| Status | When |
|---|---|
| `404` | No user matches. |
| `400` | Realm not mapped to a tenant, or `id` not parseable. |
| `401` | Missing / invalid SPI shared secret. |
