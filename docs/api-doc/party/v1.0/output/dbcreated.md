name: dbcreated
type: DbCreationInfo (json) + side-effect: a new database physically exists on the target server
trigger: [tenantProvisioning](../trigger/tenantProvisioning.md)  // can be multiple triggers later

Records a per-tenant database that was created during provisioning. One trigger run can produce more than one of these (e.g. the MariaDB projection DB and the orchestrix Postgres app DB). Captured shape:

- `engine` — `mariadb` | `postgres`
- `host`, `port`
- `name` — the new database name
- `templateSource` — null for the projection DB; the bundled artifact path for app DBs cloned from a template
- `status` — `CREATED` | `LOADED` | `FAILED`
