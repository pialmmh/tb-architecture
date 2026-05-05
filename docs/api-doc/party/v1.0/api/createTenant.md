# createTenant()

HTTP API call. Creates a new tenant and provisions everything needed for it to exist.

The owning **operator is not an API parameter** — it is bound to the JVM at startup via YAML (`application.properties` operator selector + `config/operators/<operator>/<profile>/profile-<profile>.yml`). Each Party deployment serves exactly one operator; the operator id cannot be smuggled in by a caller. This is tight coupling by design (security boundary).

## Params

- `shortName` (string, required) — slug; must match `^[a-z0-9][a-z0-9_]{0,39}$`. Used to derive DB and realm names.
- `fullName` (string, required) — human-readable name.
- `companyName` (string, optional) — legal name; defaults to `fullName`.
- `appName` (string, required) — currently only `orchestrix`.

## Computed (not supplied by caller)

- `operatorId` — resolved from the JVM's bound operator (YAML).
- `dbName` — **`<shortName>_<appName>`** (e.g. `acme_orchestrix`).
  - Underscore only, no hyphen — keeps the identifier valid in SQL without quoting (Postgres + MariaDB both lowercase unquoted identifiers and reject `-`).
  - Tenant-first ordering — lets ops list all DBs of one tenant with `\l acme_*` when debugging.
  - Same name is used for both the per-tenant projection DB (MariaDB) and the per-tenant app DB (Postgres for orchestrix). They live on different servers, so the name collision is harmless and the ergonomics are better.

## Steps

Runs synchronously on the request thread (in production, a Temporal workflow; in dev with `party.temporal.bypass=true`, in-process — bypass forbidden in prod):

1. Insert `tenant` master row in `PROVISIONING` status + audit row.
2. Create per-tenant projection DB; apply Flyway baseline; seed default roles & permissions.
3. Create the per-tenant Keycloak realm.
4. Run the app-specific provisioner selected by `appName` (currently `orchestrix` — clones the bundled Postgres template DB; Odoo runs underneath orchestrix and is not a separate app).
5. Mark master row `ACTIVE`; mark audit row `SUCCESS`.

## Produces

- [output/tenant](../output/tenant.md)
- [output/dbcreated](../output/dbcreated.md) — one or many per call (projection DB + each app DB cloned)

## Idempotency & retries

Every step is idempotent. On failure earlier steps are not rolled back; rerunning the same `createTenant()` call resumes from the failure point. The audit row stays in `PROVISIONING` between attempts and only flips to `SUCCESS` after the final step completes.

## Response

Universal envelope:

```
{
  success:       bool,
  causeCode:     int,
  causeProtocol: "HTTP",
  description:   string,
  payload:       Tenant | { errors: [...] } | null
}
```

REST. `causeCode` mirrors the HTTP status. Description uses [../cause-codes.md](../cause-codes.md) override when registered.

### Success

| causeCode | protocol | description (override)        | payload |
|----------:|----------|-------------------------------|---------|
| 201       | HTTP     | tenant provisioned            | [output/tenant](../output/tenant.md) — `Tenant` json (`dbPassRef` never returned) |

### Failures

| causeCode | protocol | description (override)                                         | when it fires                                                                            |
|----------:|----------|----------------------------------------------------------------|------------------------------------------------------------------------------------------|
| 400       | HTTP     | request body or query parameters malformed                     | missing `shortName` / `fullName` / `appName`, wrong content-type, unparseable JSON       |
| 409       | HTTP     | shortName already taken — pick another                         | a `tenant` row with this `shortName` already exists in master                            |
| 422       | HTTP     | validation failed — see `payload.errors[]`                     | `shortName` does not match `^[a-z0-9][a-z0-9_]{0,39}$`, or `appName != "orchestrix"`     |
| 500       | HTTP     | unhandled error — see server logs (request id in response)     | provisioner threw mid-run — master row may be in `PROVISIONING`; retry is safe (idempotent trigger) |
| 503       | HTTP     | upstream dependency unreachable (Postgres / MariaDB / Keycloak)| dependency check failed before any state was mutated                                     |

Every `(causeCode, protocol)` pair above is registered in [../cause-codes.md](../cause-codes.md).

### Notes

- Retry semantics: `500` after partial provisioning is not a hard failure — the provisioning steps are idempotent (see `## Steps` above) and resume from the failure point on the next `createTenant()` call with the same `shortName`. Callers should retry with backoff before treating this as a hard error.
- `409` and `422` are caller-fixable; `500`/`503` are operator-side.
