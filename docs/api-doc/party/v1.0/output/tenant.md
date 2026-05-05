name: tenant
type: Tenant (json) + master DB row (`party_master.tenant`)
produced by: [api/createTenant](../api/createTenant.md)

The freshly-provisioned tenant — returned in the response body of `createTenant()` and persisted as a row in master. Connection coordinates (`dbHost`, `dbPort`, `dbName`, `dbUser`) are populated; `dbPassRef` is write-only and never returned in JSON. State machine: `PROVISIONING → ACTIVE` on success.
