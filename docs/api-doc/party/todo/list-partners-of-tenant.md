# List all partners belonging to a tenant

**Method:** `GET`
**Path:** `/tenants/{tenantId}/partners`
**Resource group:** Partner
**Idempotent:** Yes

## Overview

Returns every `Partner` row whose `tenantId` matches the path parameter, ordered by id ascending. A "partner" in Party is a billable counter-party of the tenant (a customer, supplier, or reseller). All `auth_user` rows hang off a partner.

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `tenantId` | Long | yes | Owning tenant. |

## Query parameters

_None._

## Request body

_None._

## Response — `200 OK`

```json
[
  {
    "id": 7,
    "tenantId": 42,
    "partnerName": "Big Buyer Co.",
    "alternateNameInvoice": "BBC Ltd.",
    "address1": "12 Some Road",
    "city": "Dhaka",
    "country": "BD",
    "telephone": "+88012345678",
    "email": "billing@bbc.example",
    "customerPrepaid": false,
    "partnerType": "CUSTOMER",
    "billingDate": 1,
    "allowedDaysForInvoicePayment": 30,
    "timezoneOffsetMinutes": 360,
    "defaultCurrency": 50,
    "status": "ACTIVE",
    "createdAt": "2026-04-23T01:00:00.000Z",
    "updatedAt": "2026-04-23T01:00:00.000Z"
  }
]
```

## Errors

| Status | When |
|---|---|
| `404` | No tenant with that id. |

## Notes

- `partnerType` is a free-form enum string today (e.g. `CUSTOMER`, `SUPPLIER`, `RESELLER`).
- See [get-partner-extra-info.md](get-partner-extra-info.md) for the optional 1:1 `partner_extra` row.
