# tenantProvisioning

Named business action: stand up everything a new tenant needs to exist.

## Activated by

- input: [input/api/createTenant](../input/api/createTenant.md)
- (later) clock / re-provision schedule — TBD
- (later) chain from a parent trigger — TBD

## What it does

Runs the provisioning steps in order, synchronously on the activating thread:

1. Insert `tenant` master row in `PROVISIONING` status + audit row.
2. Create per-tenant projection DB; apply Flyway baseline; seed default roles & permissions.
3. Create the per-tenant Keycloak realm.
4. Run the app-specific provisioner selected by `appName` (currently `orchestrix` — clones the bundled Postgres template DB; Odoo runs underneath orchestrix and is not a separate app).
5. Mark master row `ACTIVE`; mark audit row `SUCCESS`.

## Produces

- [output/tenant](../output/tenant.md)
- [output/dbcreated](../output/dbcreated.md)

## Idempotency & retries

Every step is idempotent. On failure, earlier steps are not rolled back; rerunning the same `createTenant()` call resumes from the failure point. The audit row stays in `PROVISIONING` between attempts and only flips to `SUCCESS` after the final step completes.

## Notes

- In production this trigger is run by a Temporal workflow. In dev (with `party.temporal.bypass=true`) it runs in-process on the request thread. Bypass is forbidden in prod.
