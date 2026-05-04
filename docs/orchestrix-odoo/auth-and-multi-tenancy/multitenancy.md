# Orchestrix v2 — Odoo Multi-Tenancy

How tenants are isolated, resolved, and routed across the React UI, the
Spring Boot API gateway, and Odoo 19.

> Source paths in this doc are relative to
> `/home/mustafa/telcobright-projects/orchestrix-v2/`.

---

## TL;DR

- **One Postgres cluster** (port `5433`), **one Odoo process** (port `7170`).
- **Two tenancy modes**, picked by config (no code change):
  - **SINGLE** (current default) — all tenants share `odoo_billing_19`;
    isolation is per-row via Odoo's native `company_id` (multi-company).
  - **PER_DB** — each tenant gets its own DB (`odoo_<slug>`), spawned from
    `db-templates/pristine-tenant-v19.sql.gz`.
- Tenant context flows: **URL path → React → `X-Tenant-Slug` header →
  Spring → DB selection → Odoo XML-RPC**.
- A small set of **platform models** (`rbac.*`, `platform.*`, `infra.*`,
  `artifact.*`, `kb.*`, `doc.*`, plus a couple of tax/rate models) always
  resolve to the default DB regardless of the tenant header.
- The tenant registry itself is an Odoo model: `platform.tenant.config`.

---

## Topology

```
   ┌──────────────┐       ┌────────────────────┐       ┌──────────────────────┐
   │ React (5180) │ ───▶  │ APISIX (9081)      │ ───▶  │ Spring Boot (8180)   │
   │ /btcl/...    │       │  /api/odoo/*       │       │  /api/odoo/{m}/{f}   │
   └──────────────┘       │  + Keycloak OIDC   │       │  XML-RPC client      │
                          └────────────────────┘       └──────────┬───────────┘
                                                                  │
                                                                  ▼
                                                       ┌──────────────────────┐
                                                       │ Odoo 19 (7170)       │
                                                       │ Postgres 5433        │
                                                       │  • odoo_billing_19   │ ← default / SINGLE-mode
                                                       │  • odoo_<slug> ...   │ ← only in PER_DB mode
                                                       └──────────────────────┘
```

Service launcher: `launch-all.sh` (Postgres, MySQL, etcd, Kafka, Odoo,
Keycloak, Spring API, APISIX, Kill Bill, Vault, React UI).

---

## Two tenancy modes (one code path)

The Spring gateway picks the DB via `OdooProperties.dbFor(slug)`:

| Mode      | Resolution                            | DB layout                    | Isolation              |
|-----------|---------------------------------------|------------------------------|------------------------|
| `SINGLE`  | always `odoo_billing_19`              | one DB                       | row-level via `company_id` |
| `PER_DB`  | `dbPrefix + slug` (e.g. `odoo_btcl`)  | one DB per tenant            | physical (separate DB) |

Configured in `api/.../OdooProperties.java`:

```java
private TenantMode tenantMode = TenantMode.SINGLE;   // flip to PER_DB for full isolation
private String db        = "odoo_billing_19";        // default DB / SINGLE-mode DB
private String dbPrefix  = "odoo_";                  // PER_DB: dbPrefix + slug → per-tenant DB
```

**Why SINGLE is the current default** — the v17→v19 in-place migration of
the per-tenant DBs proved fragile (TOTP / JSON column drift, stale browser
cookies hitting v17-schema DBs and crashing on missing columns). The
pristine v19 template + `pristine-tenant-v19.sql.gz` exists so PER_DB can
be re-enabled without code changes once we're past the migration window.

The Odoo `dbfilter = ^odoo_billing_19$` in `odoo-backend-19/odoo.conf`
pins every Odoo HTTP request to the default DB — defence-in-depth against
stale cookies in SINGLE mode. **Flip this to a permissive pattern when
switching to PER_DB.**

---

## Request lifecycle (the tenant header pipeline)

### 1. React derives the slug from the URL path

`ui/src/services/odoo.js`:

```js
function tenantSlug() {
  const seg = (window.location?.pathname || '').split('/').filter(Boolean)[0];
  if (!seg || seg === 'login' || seg === 'callback' || seg === '404') return null;
  return seg;
}

api.interceptors.request.use(async (config) => {
  if (token)            config.headers.Authorization   = `Bearer ${token}`;
  const slug = tenantSlug();
  if (slug)             config.headers['X-Tenant-Slug'] = slug;
  return config;
});
```

So `https://app/btcl/products` → header `X-Tenant-Slug: btcl`.

### 2. APISIX gates the call (Keycloak OIDC) and proxies to Spring

`apisix/setup-routes.sh` defines:

| Route                                  | Auth        |
|----------------------------------------|-------------|
| `/api/odoo/health`                     | public      |
| `/api/odoo/res.partner/*`              | public (tenant list pre-login) |
| `/api/odoo/platform.tenant.config/*`   | public (branding pre-login)    |
| `/api/odoo/*`                          | Keycloak JWT (`openid-connect` plugin) |

### 3. Spring `TenantResolver` validates and resolves

`api/.../TenantResolver.java`:

1. Read `X-Tenant-Slug` header. Validate against
   `[a-z0-9][a-z0-9_-]{0,62}` — invalid → 403.
2. If a JWT is present, the slug **must** appear in the JWT's
   `allowed_tenants` claim, **unless** the user has `super_admin` in
   `roles` — bypass.
3. Convert to a DB name via `OdooProperties.dbFor(slug)`.

### 4. `OdooProxyController` decides default-vs-tenant DB

`api/.../OdooProxyController.java`:

```java
if (props.isPlatformModel(model)) db = props.getDb();   // platform-wide → default DB
else                              db = tenants.resolveDb(req);
Object result = odoo.call(db, model, method, args, kwargs);
```

Then forwards via XML-RPC to Odoo at `http://127.0.0.1:7170` with the
chosen DB. No per-model wrappers — adding an Odoo model is **zero Java
changes**.

---

## Platform models vs tenant models

Tenant header is **ignored** (default DB used) for these prefixes —
`OdooProperties.platformModelPrefixes`:

```
rbac.*           — RBAC roles, permissions, URL patterns
platform.*       — tenant registry (platform.tenant.config), platform-wide config
infra.*          — datacenters, racks, hosts
artifact.*       — deploy templates / artifact registry
kb.*             — Kill Bill linkage
doc.*            — document store
product.tax.rate
product.rate.history
```

Rationale: these are platform-wide registries the React shell needs *before*
any tenant context exists (e.g. the tenant-selector page calls
`platform.tenant.config.get_all_active`).

Everything else (sales, products, partners, accounting, stock, CRM…) is
tenant-scoped and obeys `X-Tenant-Slug`.

---

## The tenant registry: `platform.tenant.config`

Defined in `odoo-backend-19/custom-addons/platform_config/models/tenant_config.py`.

```
platform.tenant.config
├── partner_id              → res.partner (the company)
├── slug                    UNIQUE   ← URL path, header value, DB suffix
├── is_active
├── kb_api_key / secret     ← Kill Bill tenant credentials
├── currency, timezone
├── login_title / subtitle / app_name / app_short_name / theme   ← branding
└── overdue_warning_days / suspend_days / disconnect_days        ← policy
```

API methods exposed to React:

| Method                  | Returns                                         |
|-------------------------|-------------------------------------------------|
| `get_all_active()`      | active tenants as `[{slug, partnerId, branding, billing, overdue}, ...]` |
| `get_by_slug(slug)`     | single tenant config dict                       |

Seed (post-install hook in `platform_config/__init__.py`): `BTCL` (active),
`Telcobright` (active), `ABC ISP` (inactive). Add new tenants via the Odoo
UI — no code change needed.

---

## How the React UI consumes it

`ui/src/context/TenantContext.jsx`:

```js
async function fetchTenantConfigs() {
  return await call('platform.tenant.config', 'get_all_active', []);
}
```

Then:

1. `AuthContext` derives `allowedTenantSlugs` from the JWT
   (`super_admin` → `null` = sees all; otherwise the JWT's
   `allowed_tenants` claim).
2. `TenantContext` filters the registry by `allowedTenantSlugs`.
3. `TenantSelector.jsx` renders the cards. If the user has exactly one
   tenant, it auto-redirects to `/<slug>/`.
4. From there, every API call carries `X-Tenant-Slug: <slug>`.

---

## Spawning a new tenant

### SINGLE mode (current)

1. Create the company in Odoo: `Settings → Companies → New` (this is the
   `res.partner` that `platform.tenant.config.partner_id` will point at).
2. Create the tenant registry row: `Platform Config → Tenant Configs → New`
   (slug, branding, KB creds, overdue policy).
3. Issue a Keycloak user with `allowed_tenants: ["<slug>"]` (or
   `roles: ["super_admin"]` for cross-tenant access).
4. Done — the React shell will list the new tenant on next refresh.

### PER_DB mode

Same three steps, **plus** spawn the DB up-front. From
`odoo-backend-19/db-templates/README.md`:

```bash
TENANT=acme
PGHOST=/run/postgresql ; PGPORT=5433 ; PGUSER=mustafa

psql -h $PGHOST -p $PGPORT -U $PGUSER -d postgres \
    -c "DROP DATABASE IF EXISTS odoo_${TENANT};"
createdb -h $PGHOST -p $PGPORT -U $PGUSER odoo_${TENANT}

gunzip -c odoo-backend-19/db-templates/pristine-tenant-v19.sql.gz | \
    psql -h $PGHOST -p $PGPORT -U $PGUSER -d odoo_${TENANT} >/dev/null

psql -h $PGHOST -p $PGPORT -U $PGUSER -d odoo_${TENANT} \
    -c "UPDATE res_company SET name='${TENANT^}' WHERE id=1;"
```

The pristine template ships with the Odoo modules + custom addons already
installed and **no operational data**. To regen it, see the same README.

---

## v17 → v19 migration (one-time)

Scripts under `odoo-backend-19/tools/`:

| Script                                      | Phase                                            |
|---------------------------------------------|--------------------------------------------------|
| `migrate_phase1_foundation.py`              | partners, payment terms, tenant config, RBAC, deploy templates, leads |
| `migrate_phase2_infra_fiscal.py`            | infra rows + fiscal positions                    |
| `migrate_phase3_moves_orders.py`            | account moves + sale orders                      |
| `migrate_phase3_fixup.py`                   | post-migration cleanup                           |
| `migrate_telecom_billing_v17_to_v19.py`     | telecom-specific billing artefacts               |

Each opens psycopg2 connections to both `odoo_billing` (v17) and
`odoo_billing_19` (v19), runs in a single transaction, rolls back on
error. Schemas drift across majors → these use **column intersection**
(`cols_intersect`) rather than `SELECT *`.

Once migration is complete and stable, flipping `TenantMode → PER_DB` and
re-spawning per-tenant DBs from the pristine template is a config-only
change.

---

## Branding & theming under multi-tenancy

Two layers:

1. **Backend skin (Odoo)** — `tb_fluent_theme` addon. Applies our Fluent
   UI v9 brand ramp + Inter font globally. **One theme for all tenants in
   the Odoo backend.** Tenant-specific overrides not currently exposed.
2. **React shell** — `platform.tenant.config.branding` (theme color
   selection: green / blue / red / gray / orange / light variants) +
   `login_title` / `app_name` etc. Drives `TenantContext` + `ThemeContext`.

Per project convention (CLAUDE memory): **tenant overrides cover primary
+ sidebar only** — semantic colors stay universal.

---

## Auth & authorization

- **Identity**: Keycloak — `keycloak` realm, OIDC via APISIX
  `openid-connect` plugin.
- **Per-tenant authorization**: JWT carries `allowed_tenants: [...]`
  claim. Enforced in two places:
  1. UI: `AuthContext.allowedTenantSlugs` filters the tenant list.
  2. Server: `TenantResolver.validateAgainstJwt(slug)` rejects requests
     whose `X-Tenant-Slug` is not in the claim (super_admin bypasses).
- Header validation regex: `[a-z0-9][a-z0-9_-]{0,62}` — defends against
  injection / DB-name shenanigans.

---

## Known caveats

1. **`dbfilter = ^odoo_billing_19$`** is hard-coded for SINGLE mode. Must
   change to a permissive pattern (e.g. `^odoo_.*$`) before flipping to
   PER_DB.
2. **Stale browser cookies** carrying old v17 DB names crashed v19 with
   `column res_users.totp_last_counter does not exist` — the dbfilter
   above is the fix. Keep it until all clients have rotated.
3. **`res.partner` is shared** in SINGLE mode. Tenant separation relies
   on every tenant-scoped record having `company_id` set correctly.
   Cross-tenant queries that ignore `company_id` will leak.
4. **Tenant cache**: `TenantContext._tenantConfigCache` is in-memory and
   does not auto-invalidate on backend changes — refresh required.
5. **Public APISIX routes** (`res.partner/*`, `platform.tenant.config/*`)
   are world-readable so the login page can render branding before auth.
   Don't put secrets in those models.

---

## File index (quick reference)

| Concern                | Path                                                                       |
|------------------------|----------------------------------------------------------------------------|
| Odoo config            | `odoo-backend-19/odoo.conf`                                                |
| Custom addons          | `odoo-backend-19/custom-addons/`                                           |
| Tenant registry model  | `odoo-backend-19/custom-addons/platform_config/models/tenant_config.py`    |
| Pristine DB template   | `odoo-backend-19/db-templates/pristine-tenant-v19.sql.gz`                  |
| v17→v19 migrators      | `odoo-backend-19/tools/migrate_phase*.py`                                  |
| Spring tenant resolver | `api/src/main/java/com/telcobright/api/tenant/TenantResolver.java`         |
| Spring DB selector     | `api/src/main/java/com/telcobright/api/config/OdooProperties.java`         |
| Odoo proxy controller  | `api/src/main/java/com/telcobright/api/controller/OdooProxyController.java`|
| APISIX routes          | `apisix/setup-routes.sh`                                                   |
| React tenant context   | `ui/src/context/TenantContext.jsx`                                         |
| React odoo client      | `ui/src/services/odoo.js`                                                  |
| Tenant selector page   | `ui/src/pages/TenantSelector.jsx`                                          |
| Service launcher       | `launch-all.sh`                                                            |
