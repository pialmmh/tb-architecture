# Party Service — Inputs & Outputs Map

Bird's-eye view of every input the Party service consumes and every output it produces. Each output is keyed to the **trigger(s)** that cause it (one trigger may produce many outputs; one output may be produced by many triggers). The per-endpoint API docs in the same folder stay as the detailed reference; this file is the contract surface.

---

## party
### input
- **api:**  (HTTP REST under `/api/v1`)
  - **operator lifecycle**
    - `listOperators()`
    - `getOperator(id)`
    - `createOperator(body)`
    - `updateOperator(id, patch)`
    - `deleteOperator(id)`
  - **operator-user lifecycle**
    - `listOperatorUsers()`
    - `getOperatorUser(id)`
    - `createOperatorUser(body)`
    - `resetOperatorUserPassword(id, body)`
    - `setOperatorUserStatus(id, body)`
    - `deleteOperatorUser(id)`
  - **tenant lifecycle** *(marquee = provisionTenant)*
    - `listTenantsOfOperator(operatorId)`
    - `provisionTenant(operatorId, body{shortName, fullName, companyName, appName, …})`  ← runs full pipeline synchronously
    - `getTenant(id)`
    - `updateTenant(id, patch)`
    - `deleteTenant(id)`
  - **partner lifecycle**
    - `listPartnersOfTenant(tenantId)`
    - `getPartner(tenantId, id)`
    - `createPartner(tenantId, body)`
    - `updatePartner(tenantId, id, patch)`
    - `deletePartner(tenantId, id)`
    - `getPartnerExtra(tenantId, id)`
    - `upsertPartnerExtra(tenantId, id, body)`
  - **auth-user lifecycle**
    - `listAuthUsersOfTenant(tenantId)`
    - `listAuthUsersOfPartner(tenantId, partnerId)`
    - `createAuthUser(tenantId, partnerId, body)`
    - `getAuthUser(tenantId, id)`
    - `updateAuthUser(tenantId, id, patch)`
    - `resetAuthUserPassword(tenantId, id, body)`
    - `setAuthUserRoles(tenantId, id, body{roleIds})`
    - `deleteAuthUser(tenantId, id)`
    - `listUserIpRules(tenantId, userId)`
    - `addUserIpRule(tenantId, userId, body)`
    - `deleteUserIpRule(tenantId, userId, ruleId)`
    - `listUserMenuPermissions(tenantId, userId)`
    - `upsertUserMenuPermission(tenantId, userId, body)`
  - **role + permission lifecycle**
    - `listRolesOfTenant(tenantId)`
    - `getRole(tenantId, id)`
    - `createRole(tenantId, body)`
    - `updateRole(tenantId, id, patch)`
    - `deleteRole(tenantId, id)`
    - `setRolePermissions(tenantId, id, body{permissionIds})`
    - `listPermissionsOfTenant(tenantId)`
    - `createPermission(tenantId, body)`
    - `deletePermission(tenantId, id)`
  - **audit / sync visibility**
    - `listTenantSyncJobs(tenantId, ?status, ?limit)`
    - `getSyncJob(tenantId, id)`
  - **authentication (legacy Party-issued JWT)**
    - `login(body{email, password})`
  - **keycloak SPI federation (LAN-only `/internal/kc/*`)**
    - `kcFindUserByUsername(?realm, ?username)`
    - `kcFindUserById(?realm, ?id)`
    - `kcSearchUsers(?realm, ?q, ?first, ?max)`
    - `kcValidateCredentials(body{realm, username, password})`
    - `kcHealthz()`
  - **service liveness**
    - `ping()`

- **kafka:**  (planned — federation phase 1)
  - `topic: party.federation.inbound.<system_id>`  →  envelope `{source, system_id, tenant_id, external_id, version, op, payload}` for user/partner/role created/updated/deleted events from Odoo, Plane, CRM, etc. Idempotency: drop if `source==party`; drop if `version <= existing`.

- **config:**  (filesystem + classpath, loaded once at startup)
  - `application.properties`  ← bootstrap stub: operator selector + Quarkus essentials
  - `config/operators/<operator>/<profile>/profile-<profile>.yml`  ← runtime knobs (datasource, Temporal target, JWT secret refs, Keycloak admin coords, app provisioner config)
  - `db/migration/master/V*.sql`  ← Flyway master schema
  - `db/migration/tenant/V*.sql`  ← Flyway per-tenant projection schema
  - `artifact/templates/orchestrix.sql.gz`  ← bundled Postgres template DB (pg_dump of the orchestrix-on-Odoo reference DB) loaded into each new tenant's app DB

- **env / secrets:**  (resolved by `PartyProfileConfigSource` and Quarkus config)
  - `PARTY_OPERATOR_NAME`, `PARTY_OPERATOR_PROFILE`  ← which (operator, profile) this JVM bootstraps as
  - `PARTY_DB_PASSWORD`  ← master DB connection password
  - `PARTY_JWT_SECRET`  ← HS256 signing key for legacy JWT
  - `PARTY_KC_INTEGRATION_SECRET`  ← shared secret for `/internal/kc/*` SPI calls
  - `PARTY_KC_ADMIN_URL`, `PARTY_KC_ADMIN_USER`, `PARTY_KC_ADMIN_PASS`  ← Keycloak admin REST coords (only when `party.keycloak.admin.enabled=true`)
  - `PARTY_TENANT_DB_PASS`  ← password used by tenant-projection writers to connect to the per-tenant MariaDB schema
  - `PARTY_TEMPORAL_TARGET`, `PARTY_TEMPORAL_NAMESPACE`  ← when `party.temporal.bypass=false`

### output
- name: `operator`
  type: `Operator` (json) + DB row (`party_master.operator`)
  trigger: `createOperator()`, `updateOperator()`, `deleteOperator()`

- name: `operator_user`
  type: `OperatorUser` (json) + DB row (`party_master.operator_user`)
  trigger: `createOperatorUser()`, `updateOperator…`, `resetOperatorUserPassword()`, `setOperatorUserStatus()`, `deleteOperatorUser()`

- name: `tenant`
  type: `Tenant` (json) + DB row (`party_master.tenant`)
  trigger: `provisionTenant()`, `updateTenant()`, `deleteTenant()`

- name: `tenant_provisioning_pipeline`
  type: composite (synchronous fan-out — see components below)
  trigger: `provisionTenant()` — single trigger, six observable side-effects
  components:
  - name: `tenant_master_row`
    type: DB row (`party_master.tenant`), state machine `PROVISIONING → ACTIVE`
  - name: `tenant_sync_job_provision_row`
    type: DB row (`party_master.tenant_sync_job`, `entityType=TENANT_PROVISION`, `operation=PROVISION`)
  - name: `tenant_projection_db`
    type: MariaDB schema `{opShort}_{opId}_{tnShort}_{tnId}` created + Flyway-baselined + seeded with default roles & permissions
  - name: `keycloak_realm`
    type: Keycloak realm `tenant-<shortName>` provisioned via admin REST + User-Storage SPI federation provider configured pointing back at `/internal/kc/*`
  - name: `app_db`  *(currently only orchestrix)*
    type: Postgres database `orchestrix_<shortName>` cloned from the bundled `orchestrix.sql.gz` template; `res_company.name` patched to `tenant.companyName`
  - name: `audit_completion`
    type: `tenant_sync_job` row updated to `SUCCESS` or `FAILED` (with error text + duration) before the HTTP response returns

- name: `partner`
  type: `Partner` (json) + DB row (`party_master.partner`)
  trigger: `createPartner()`, `updatePartner()`, `deletePartner()`

- name: `partner_extra`
  type: `PartnerExtra` (json) + DB row (`party_master.partner_extra`)
  trigger: `upsertPartnerExtra()`

- name: `auth_user`
  type: `AuthUser` (json, never `passwordHash`) + DB row (`party_master.auth_user`)
  trigger: `createAuthUser()`, `updateAuthUser()`, `resetAuthUserPassword()`, `deleteAuthUser()`

- name: `user_role_assignment`
  type: rows in (`party_master.user_role`) + projection mirror
  trigger: `createAuthUser(roleIds)`, `setAuthUserRoles()`

- name: `auth_role`
  type: `AuthRole` (json) + DB row (`party_master.auth_role`)
  trigger: `createRole()`, `updateRole()`, `deleteRole()`

- name: `role_permission_assignment`
  type: rows in (`party_master.role_permission`) + projection mirror
  trigger: `setRolePermissions()`

- name: `auth_permission`
  type: `AuthPermission` (json) + DB row (`party_master.auth_permission`)
  trigger: `createPermission()`, `deletePermission()`

- name: `ip_access_rule`
  type: `IpAccessRule` (json) + DB row (`party_master.ip_access_rule`)
  trigger: `addUserIpRule()`, `deleteUserIpRule()`

- name: `ui_menu_permission`
  type: `UiMenuPermission` (json) + DB row (`party_master.ui_menu_permission`)
  trigger: `upsertUserMenuPermission()`

- name: `tenant_sync_job`
  type: DB row (`party_master.tenant_sync_job`) — audit-trail entry
  trigger: every mutating API call (`provisionTenant()`, all `create*()`, `update*()`, `delete*()`, `set*()`, `upsert*()`, `addUserIpRule()`, …)

- name: `tenant_projection_write`
  type: row write into the per-tenant MariaDB schema (mirror of master change)
  trigger: dispatch of any non-`PROVISION` `tenant_sync_job` — one-for-one mirror of the upstream master mutation

- name: `temporal_workflow_started`
  type: Temporal workflow execution (`ProvisionTenantWorkflow` or `SyncEntityWorkflow`)
  trigger: any dispatch when `party.temporal.bypass=false`. **Note:** bypass=true is dev-only; production MUST run with bypass=false.

- name: `jwt_pair`
  type: `{accessToken, refreshToken, expiresInSeconds, tenantId, userId}` (json)
  trigger: `login()`

- name: `auth_user_last_login_update`
  type: DB column write (`auth_user.last_login_at`)
  trigger: `login()` (success path)

- name: `kc_user_view`
  type: `KcUserView` (json) — Keycloak-shaped user record served back to the User Storage SPI
  trigger: `kcFindUserByUsername()`, `kcFindUserById()`, `kcSearchUsers()`

- name: `kc_credentials_validation`
  type: `{valid: bool}` (json)
  trigger: `kcValidateCredentials()`

- name: `external_user_event_outbound`  *(planned — federation phase 1)*
  type: Kafka envelope (json) on `topic: party.federation.outbound.<entity>`
  trigger: `createAuthUser()`, `updateAuthUser()`, `deleteAuthUser()`, `createPartner()`, `updatePartner()`, … — every mutation that should fan out to registered external systems (Odoo, Plane, CRM, …)
  consumed_by: each registered `ExternalSystemAdapter` (one consumer per `system_id`)

- name: `structured_log_line`
  type: log line (stdout in dev, file `/var/log/party/party.log` in prod, also captured by systemd journal)
  trigger: every request entry/exit, every Temporal activity start/finish, every error, every `[PartyProfileConfigSource]` config-load summary

- name: `prometheus_metrics`
  type: Prometheus text exposition format
  trigger: scrape on `GET /q/metrics`

- name: `health_status`
  type: `{status: UP|DOWN, checks: [...]}` (json)
  trigger: `GET /q/health`, `GET /q/health/live`, `GET /q/health/ready`

- name: `flyway_schema_history`
  type: DB rows (`party_master.flyway_schema_history`) recording each applied migration
  trigger: app startup with `quarkus.flyway.migrate-at-start=true`

- name: `version_file`
  type: file `/opt/party/version.txt` (operator, profile, commit, branch, deployedAt, deployedBy)
  trigger: `tools/deploy/remote-deploy.sh` (deploy host writes to remote — not produced by the JVM itself, kept here for completeness)

- name: `deploy_history_log`
  type: append-only log file `/opt/party/deploy-history.log`
  trigger: every successful run of `tools/deploy/remote-deploy.sh`
