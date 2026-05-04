# Create a new partner under a tenant

**Method:** `POST`
**Path:** `/tenants/{tenantId}/partners`
**Resource group:** Partner
**Idempotent:** No (each call creates a new row).

## Overview

Inserts a new `Partner` under the tenant and enqueues a `tenant_sync_job` so the per-tenant projection DB receives the same row. Use this before creating any `auth_user` rows for the partner.

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `tenantId` | Long | yes | Owning tenant. |

## Query parameters

_None._

## Request body

```json
{
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
  "defaultCurrency": 50
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `partnerName` | string ≤ 120 | yes | Display name. |
| `alternateNameInvoice` | string ≤ 400 | no | Name shown on invoices. |
| `alternateNameOther` | string ≤ 400 | no | Alternate name for other documents. |
| `address1` / `address2` / `city` / `state` / `postalCode` / `country` | string | no | Address fields. |
| `telephone` | string ≤ 45 | no | Phone. |
| `email` | string ≤ 120 | no | Email. |
| `customerPrepaid` | bool | yes | `true` if billed up front, `false` if invoiced. |
| `partnerType` | string ≤ 30 | yes | `CUSTOMER` / `SUPPLIER` / `RESELLER` / etc. |
| `billingDate` | int | no | Day-of-month invoices are cut. |
| `allowedDaysForInvoicePayment` | int | no | Net-N payment terms. |
| `timezoneOffsetMinutes` | int | no | Tenant-side display TZ offset. |
| `callSrcId` | int | no | Call-source identifier (telco-specific). |
| `defaultCurrency` | int | yes | Currency id (numeric, references a currency table). |
| `invoiceAddress` | string | no | Address override for invoices. |
| `vatRegistrationNo` | string | no | VAT registration number. |
| `paymentAdvice` | string | no | Free-text payment instructions. |

## Response — `200 OK`

The created `Partner` (status defaults to `ACTIVE`).

## Errors

| Status | When |
|---|---|
| `400` | Missing required field. |
| `404` | No tenant with that id. |

## Notes

- Side effect: a `tenant_sync_job` row with `entityType=PARTNER`, `operation=INSERT`.
- For the optional extended-info row, see [upsert-partner-extra-info.md](upsert-partner-extra-info.md).
