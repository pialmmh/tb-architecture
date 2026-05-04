# Upsert a partner's extended info

**Method:** `PUT`
**Path:** `/tenants/{tenantId}/partners/{id}/extra`
**Resource group:** Partner
**Idempotent:** Yes

## Overview

Creates or updates the `partner_extra` row attached to a partner. PUT semantics: the row is fully replaced by the supplied body (fields not in the body are NULLed). For partial-update semantics, fetch first via [get-partner-extra-info.md](get-partner-extra-info.md), merge, then PUT.

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `tenantId` | Long | yes | Owning tenant. |
| `id` | Long | yes | Partner primary key. |

## Query parameters

_None._

## Request body

```json
{
  "address1": "12 Some Road",
  "address2": "Apt 4B",
  "city": "Dhaka",
  "state": "Dhaka Division",
  "postalCode": "1212",
  "nid": "1234567890123",
  "tradeLicense": "TL-99999",
  "tin": "TIN-12345",
  "countryCode": "BD"
}
```

| Field | Type | Description |
|---|---|---|
| `address1` / `address2` / `address3` / `address4` | string ≤ 200 | Address lines. |
| `city` | string ≤ 80 | City. |
| `state` | string ≤ 80 | State / division. |
| `postalCode` | string ≤ 40 | Postal code. |
| `nid` | string ≤ 40 | National ID. |
| `tradeLicense` | string ≤ 80 | Trade license number. |
| `tin` | string ≤ 40 | Tax identification number. |
| `taxReturnDate` | date (`YYYY-MM-DD`) | Last tax-return filing date. |
| `countryCode` | string ≤ 5 | ISO country code. |

## Response — `200 OK`

The full `PartnerExtra` row (same shape as [get-partner-extra-info.md](get-partner-extra-info.md)).

## Errors

| Status | When |
|---|---|
| `404` | No partner with that `(tenantId, id)` pair. |

## Notes

- Side effect: a `tenant_sync_job` row with `entityType=PARTNER_EXTRA`, `operation=UPSERT`.
