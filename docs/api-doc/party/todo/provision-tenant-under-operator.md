# Provision a new tenant under an operator

**Method:** `POST`
**Path:** `/operators/{operatorId}/tenants`
**Resource group:** Tenant lifecycle
**Idempotent:** Yes (re-running the same call with the same `shortName` resumes provisioning instead of duplicating side-effects).

## Overview

Creates a new `Tenant` row under the given operator and runs the **full provisioning pipeline synchronously** before returning. The HTTP response only comes back after every step has either succeeded or thrown — there is no fire-and-forget queueing. A successful response carries the final `Tenant` object with `status="ACTIVE"`.

The provisioning pipeline performs, in order, on the request thread:

1. Insert master row (`tenant.status = "PROVISIONING"`) and an audit `tenant_sync_job` row (also `PROVISIONING`).
2. Create the per-tenant MariaDB projection database (`{opShort}_{opId}_{tnShort}_{tnId}`).
3. Apply the tenant Flyway baseline + seed default roles & permissions.
4. Create a per-tenant Keycloak realm.
5. Run the **app-specific provisioner** selected by `appName` (currently only `orchestrix`, which clones the bundled Postgres template DB for the tenant — Odoo runs underneath orchestrix and is not exposed as a separate app).
6. Update master row to `ACTIVE`, audit row to `SUCCESS`.

If any step throws, earlier steps are **not rolled back** — every step is idempotent, so the same call can be re-run to resume from the failure point.

A typical successful call takes **5–30 seconds**, dominated by the app-template DB load. Callers MUST set HTTP read timeouts ≥ 60s.

> **Production note:** the `party.temporal.bypass` flag (which routes provisioning through `LocalSyncDispatcher` instead of dispatching a Temporal workflow) is **forbidden in production profiles**. In prod the Temporal worker MUST be the executor; the synchronous bypass is a developer-laptop convenience only.

## Path parameters

| Name | Type | Required | Description |
|---|---|---|---|
| `operatorId` | `Long` | yes | The owning operator's id. Must reference an existing operator in `ACTIVE` status. |

## Query parameters

_None._

## Request body

```json
{
  "shortName": "acme",
  "fullName": "Acme Corporation Ltd.",
  "companyName": "Acme Corp",
  "appName": "orchestrix",
  "address1": "12 Some Road",
  "city": "Dhaka",
  "country": "BD",
  "phone": "+88012345678",
  "email": "ops@acme.example"
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `shortName` | string ≤ 40 chars | yes | Lowercase slug used to derive DB names + Keycloak realm. Must match `^[a-z0-9][a-z0-9_-]{0,39}$`; forbidden values collide with reserved words. Unique within an operator. |
| `fullName` | string ≤ 200 | yes | Human-readable tenant name shown in admin UIs. |
| `companyName` | string ≤ 200 | no | Legal company name; falls back to `fullName` if absent. Written into per-tenant app DBs (e.g. `res_company.name` for orchestrix/Odoo). |
| `appName` | string | yes | The single app this tenant is provisioned for. **Only `orchestrix` is currently supported.** Other values → `400 Bad Request`. |
| `address1` | string ≤ 200 | no | First address line. |
| `city` | string ≤ 80 | no | City. |
| `country` | string ≤ 80 | no | ISO-3166-1 alpha-2 or alpha-3 country code; not validated server-side. |
| `phone` | string ≤ 40 | no | Contact phone. |
| `email` | string ≤ 120 | no | Contact email. |

The server populates `id`, `operatorId`, `dbHost`, `dbPort`, `dbName`, `dbUser`, `dbPassRef`, `status`, `createdAt`, `updatedAt`. Any client-supplied values for those fields are ignored.

## Response — `200 OK`

```json
{
  "id": 42,
  "operatorId": 1,
  "shortName": "acme",
  "fullName": "Acme Corporation Ltd.",
  "companyName": "Acme Corp",
  "address1": "12 Some Road",
  "city": "Dhaka",
  "country": "BD",
  "phone": "+88012345678",
  "email": "ops@acme.example",
  "dbHost": "10.10.199.20",
  "dbPort": 3306,
  "dbName": "telcobright_1_acme_42",
  "dbUser": "tenant_acme_42",
  "status": "ACTIVE",
  "createdAt": "2026-05-05T08:14:23.211Z",
  "updatedAt": "2026-05-05T08:14:48.802Z"
}
```

`dbPassRef` is write-only (never returned). Status transitions through this call: `PROVISIONING → ACTIVE` on success, `PROVISIONING` left in place on failure.

## Errors

| Status | When |
|---|---|
| `400 Bad Request` | Missing `shortName` / `fullName` / `appName`, malformed slug, unsupported `appName`. |
| `404 Not Found` | `operatorId` does not exist. |
| `409 Conflict` | A tenant with the same `(operatorId, shortName)` already exists in `ACTIVE` status. |
| `500 Internal Server Error` | Any provisioning step threw. The body includes the failed step name and exception summary. The master `tenant` row is left in `PROVISIONING` so a retry of the same call resumes. |

Failure body shape:
```json
{ "step": "orchestrix-load-template", "error": "psql exit code 1: ..." }
```

## Notes

- **Synchronous-only contract.** No 202 / job-token semantics. The response body always reflects the final state — no polling required.
- **Single app per tenant.** The platform stance is "one tenant, one app". Multi-app tenants are out of scope; if a tenant later needs a second app, it gets a second tenant row (and a second projection DB / realm).
- **Why `orchestrix` is the only valid app today:** orchestrix sits on top of Odoo for billing & company data. The user-facing app is orchestrix; Odoo is an internal subsystem and is provisioned as part of orchestrix's actions, not exposed to clients.
- **Idempotency is by `shortName` per operator.** Re-running with the same `shortName` while a previous run is still `PROVISIONING` resumes the pipeline; once the row is `ACTIVE`, re-running returns `409`.
- **Audit trail.** A `tenant_sync_job` row with `entityType=TENANT_PROVISION`, `operation=PROVISION` is written before activities run and updated to `SUCCESS` / `FAILED` after. List with `GET /tenants/{tenantId}/sync-jobs`.
- **Related:** [list-tenants-of-operator.md](list-tenants-of-operator.md), [list-tenant-sync-jobs.md](list-tenant-sync-jobs.md), [update-tenant.md](update-tenant.md).
