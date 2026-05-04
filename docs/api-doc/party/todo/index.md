# Party Service — API Index

> **Looking for the bird's-eye contract surface?** See [inputs-and-outputs.md](inputs-and-outputs.md) — every input the service consumes (HTTP, Kafka, config, env), every output it produces (JSON, DB rows, Keycloak realms, app DBs, Temporal workflows, logs, metrics), each output keyed to the trigger(s) that cause it.

Party is the canonical owner of `operator → tenant → partner → user → role → permission → ip-rules → menu-perms`. Master data lives in the `party_master` schema (Postgres); per-tenant projections live in dedicated MariaDB schemas, kept in sync by Temporal workflows (or, in dev/laptop mode, by `LocalSyncDispatcher` running activities synchronously in-process).

**Base URL (dev):** `http://localhost:18081/api/v1`
**Base URL (prod):** `http://<host>:18081/api/v1` (overlay-internal)

All endpoints return JSON unless noted otherwise. Authentication for tenant-facing endpoints is currently `POST /auth/login` (email + password → JWT); a Keycloak-issued OIDC bearer is the target. Internal `/internal/kc/*` is LAN-only — never exposed publicly.

**Conventions in these docs:**
- One markdown file per HTTP method + path.
- Filename is a verb-phrase that reads like an English action.
- Each file: overview at top, then path / query / body / response / errors / notes.

---

## Operator (telco operator — top of the graph)

| File | Method | Path |
|---|---|---|
| [list-all-operators.md](list-all-operators.md) | GET | `/operators` |
| [get-operator-by-id.md](get-operator-by-id.md) | GET | `/operators/{id}` |
| [create-operator.md](create-operator.md) | POST | `/operators` |
| [update-operator.md](update-operator.md) | PATCH | `/operators/{id}` |
| [delete-operator.md](delete-operator.md) | DELETE | `/operators/{id}` |

## Tenant (a customer of an operator)

| File | Method | Path |
|---|---|---|
| [list-tenants-of-operator.md](list-tenants-of-operator.md) | GET | `/operators/{operatorId}/tenants` |
| [provision-tenant-under-operator.md](provision-tenant-under-operator.md) | POST | `/operators/{operatorId}/tenants` |
| [get-tenant-by-id.md](get-tenant-by-id.md) | GET | `/tenants/{id}` |
| [update-tenant.md](update-tenant.md) | PATCH | `/tenants/{id}` |
| [delete-tenant.md](delete-tenant.md) | DELETE | `/tenants/{id}` |

## Operator users (super-admins, scoped to one operator)

| File | Method | Path |
|---|---|---|
| [list-all-operator-users.md](list-all-operator-users.md) | GET | `/operator-users` |
| [get-operator-user-by-id.md](get-operator-user-by-id.md) | GET | `/operator-users/{id}` |
| [create-operator-user.md](create-operator-user.md) | POST | `/operator-users` |
| [reset-operator-user-password.md](reset-operator-user-password.md) | POST | `/operator-users/{id}/password` |
| [set-operator-user-status.md](set-operator-user-status.md) | POST | `/operator-users/{id}/status` |
| [delete-operator-user.md](delete-operator-user.md) | DELETE | `/operator-users/{id}` |

## Partner (a billable counter-party of a tenant)

| File | Method | Path |
|---|---|---|
| [list-partners-of-tenant.md](list-partners-of-tenant.md) | GET | `/tenants/{tenantId}/partners` |
| [get-partner-by-id.md](get-partner-by-id.md) | GET | `/tenants/{tenantId}/partners/{id}` |
| [create-partner-under-tenant.md](create-partner-under-tenant.md) | POST | `/tenants/{tenantId}/partners` |
| [update-partner.md](update-partner.md) | PATCH | `/tenants/{tenantId}/partners/{id}` |
| [delete-partner.md](delete-partner.md) | DELETE | `/tenants/{tenantId}/partners/{id}` |
| [get-partner-extra-info.md](get-partner-extra-info.md) | GET | `/tenants/{tenantId}/partners/{id}/extra` |
| [upsert-partner-extra-info.md](upsert-partner-extra-info.md) | PUT | `/tenants/{tenantId}/partners/{id}/extra` |

## Auth users (end-users of a tenant, owned by a partner)

| File | Method | Path |
|---|---|---|
| [list-all-auth-users-of-tenant.md](list-all-auth-users-of-tenant.md) | GET | `/tenants/{tenantId}/users` |
| [list-auth-users-of-partner.md](list-auth-users-of-partner.md) | GET | `/tenants/{tenantId}/partners/{partnerId}/users` |
| [create-auth-user-under-partner.md](create-auth-user-under-partner.md) | POST | `/tenants/{tenantId}/partners/{partnerId}/users` |
| [get-auth-user-by-id.md](get-auth-user-by-id.md) | GET | `/tenants/{tenantId}/users/{id}` |
| [update-auth-user.md](update-auth-user.md) | PATCH | `/tenants/{tenantId}/users/{id}` |
| [reset-auth-user-password.md](reset-auth-user-password.md) | POST | `/tenants/{tenantId}/users/{id}/password` |
| [set-auth-user-roles.md](set-auth-user-roles.md) | POST | `/tenants/{tenantId}/users/{id}/roles` |
| [delete-auth-user.md](delete-auth-user.md) | DELETE | `/tenants/{tenantId}/users/{id}` |
| [list-ip-access-rules-of-user.md](list-ip-access-rules-of-user.md) | GET | `/tenants/{tenantId}/users/{userId}/ip-rules` |
| [add-ip-access-rule-to-user.md](add-ip-access-rule-to-user.md) | POST | `/tenants/{tenantId}/users/{userId}/ip-rules` |
| [delete-ip-access-rule-of-user.md](delete-ip-access-rule-of-user.md) | DELETE | `/tenants/{tenantId}/users/{userId}/ip-rules/{ruleId}` |
| [list-menu-permissions-of-user.md](list-menu-permissions-of-user.md) | GET | `/tenants/{tenantId}/users/{userId}/menu-permissions` |
| [upsert-menu-permission-of-user.md](upsert-menu-permission-of-user.md) | PUT | `/tenants/{tenantId}/users/{userId}/menu-permissions` |

## Auth roles

| File | Method | Path |
|---|---|---|
| [list-roles-of-tenant.md](list-roles-of-tenant.md) | GET | `/tenants/{tenantId}/roles` |
| [get-role-by-id.md](get-role-by-id.md) | GET | `/tenants/{tenantId}/roles/{id}` |
| [create-role-under-tenant.md](create-role-under-tenant.md) | POST | `/tenants/{tenantId}/roles` |
| [update-role.md](update-role.md) | PATCH | `/tenants/{tenantId}/roles/{id}` |
| [delete-role.md](delete-role.md) | DELETE | `/tenants/{tenantId}/roles/{id}` |
| [set-permissions-of-role.md](set-permissions-of-role.md) | POST | `/tenants/{tenantId}/roles/{id}/permissions` |

## Auth permissions

| File | Method | Path |
|---|---|---|
| [list-permissions-of-tenant.md](list-permissions-of-tenant.md) | GET | `/tenants/{tenantId}/permissions` |
| [create-permission-under-tenant.md](create-permission-under-tenant.md) | POST | `/tenants/{tenantId}/permissions` |
| [delete-permission.md](delete-permission.md) | DELETE | `/tenants/{tenantId}/permissions/{id}` |

## Tenant sync jobs (audit trail of provisioning + projection writes)

| File | Method | Path |
|---|---|---|
| [list-tenant-sync-jobs.md](list-tenant-sync-jobs.md) | GET | `/tenants/{tenantId}/sync-jobs` |
| [get-sync-job-by-id.md](get-sync-job-by-id.md) | GET | `/tenants/{tenantId}/sync-jobs/{id}` |

## Authentication

| File | Method | Path |
|---|---|---|
| [login-with-email-and-password.md](login-with-email-and-password.md) | POST | `/auth/login` |

## Internal Keycloak SPI (LAN-only; consumed by the User Storage SPI)

| File | Method | Path |
|---|---|---|
| [internal-kc-find-user-by-username.md](internal-kc-find-user-by-username.md) | GET | `/internal/kc/users/by-username` |
| [internal-kc-find-user-by-id.md](internal-kc-find-user-by-id.md) | GET | `/internal/kc/users/by-id` |
| [internal-kc-search-users.md](internal-kc-search-users.md) | GET | `/internal/kc/users/search` |
| [internal-kc-validate-user-credentials.md](internal-kc-validate-user-credentials.md) | POST | `/internal/kc/users/validate-credentials` |
| [internal-kc-keycloak-spi-healthz.md](internal-kc-keycloak-spi-healthz.md) | GET | `/internal/kc/healthz` |

## Service health

| File | Method | Path |
|---|---|---|
| [ping-service-liveness.md](ping-service-liveness.md) | GET | `/ping` |
