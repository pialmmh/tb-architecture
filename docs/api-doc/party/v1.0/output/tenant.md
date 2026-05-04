name: tenant
type: Tenant (json) + master DB row (`party_master.tenant`)
trigger: [tenantProvisioning](../trigger/tenantProvisioning.md)  // can be multiple triggers later

The freshly-provisioned tenant — returned in the response body of the call that fired the trigger, and persisted as a row in master. Connection coordinates (`dbHost`, `dbPort`, `dbName`, `dbUser`) are populated; `dbPassRef` is write-only and never returned in JSON. State machine: `PROVISIONING → ACTIVE` on success.
