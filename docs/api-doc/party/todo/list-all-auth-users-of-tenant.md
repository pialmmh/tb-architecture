# List all auth users belonging to a tenant

**Method:** `GET`
**Path:** `/tenants/{tenantId}/users`
**Resource group:** Auth user
**Idempotent:** Yes

## Overview

Returns every `auth_user` row in the tenant, regardless of which partner they belong to. Useful for tenant-wide admin views. For a partner-scoped view, use [list-auth-users-of-partner.md](list-auth-users-of-partner.md).

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `tenantId` | Long | yes | Tenant primary key. |

## Query parameters

_None._

## Request body

_None._

## Response — `200 OK`

```json
[
  {
    "id": 11,
    "tenantId": 42,
    "partnerId": 7,
    "email": "user@bbc.example",
    "firstName": "Some",
    "lastName": "User",
    "phone": "+88012345678",
    "userStatus": "ACTIVE",
    "resellerDbName": null,
    "pbxUuid": null,
    "lastLoginAt": "2026-05-04T22:01:14.123Z",
    "createdAt": "2026-04-23T05:30:00.000Z",
    "updatedAt": "2026-05-04T22:01:14.123Z"
  }
]
```

`passwordHash` is `@JsonIgnore` and never returned.

## Errors

| Status | When |
|---|---|
| `404` | No tenant with that id. |

## Notes

- Pagination is not currently supported. For tenants with many users, consider filtering by partner.
