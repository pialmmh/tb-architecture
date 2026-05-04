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

## Fires trigger

- [trigger/tenantProvisioning](../../trigger/tenantProvisioning.md)
