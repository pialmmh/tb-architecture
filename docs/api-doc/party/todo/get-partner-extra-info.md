# Get a partner's extended info

**Method:** `GET`
**Path:** `/tenants/{tenantId}/partners/{id}/extra`
**Resource group:** Partner
**Idempotent:** Yes

## Overview

Fetches the optional 1:1 `partner_extra` row for a partner. Holds NID, trade license, TIN, addresses 2–4, country code, and similar enrichment fields not in the core `partner` row.

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

```json
{
  "id": 1,
  "tenantId": 42,
  "partnerId": 7,
  "address1": "12 Some Road",
  "address2": "Apt 4B",
  "address3": null,
  "address4": null,
  "city": "Dhaka",
  "state": "Dhaka Division",
  "postalCode": "1212",
  "nid": "1234567890123",
  "tradeLicense": "TL-99999",
  "tin": "TIN-12345",
  "countryCode": "BD",
  "createdAt": "2026-04-23T01:00:00.000Z",
  "updatedAt": "2026-04-23T01:00:00.000Z"
}
```

## Errors

| Status | When |
|---|---|
| `404` | No partner with that `(tenantId, id)` pair, OR no extra row exists yet (use [upsert-partner-extra-info.md](upsert-partner-extra-info.md) to create). |

## Notes

- The extra row is created lazily by the upsert call.
