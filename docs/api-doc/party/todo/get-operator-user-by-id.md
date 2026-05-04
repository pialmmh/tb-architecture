# Get an operator user by id

**Method:** `GET`
**Path:** `/operator-users/{id}`
**Resource group:** Operator user
**Idempotent:** Yes

## Overview

Fetches a single `OperatorUser` (platform super-admin scoped to one operator) by primary key.

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `id` | Long | yes | Operator-user primary key. |

## Query parameters

_None._

## Request body

_None._

## Response — `200 OK`

Same shape as the entries returned by [list-all-operator-users.md](list-all-operator-users.md).

## Errors

| Status | When |
|---|---|
| `404` | No operator-user with that id. |

## Notes

- `passwordHash` is never returned.
