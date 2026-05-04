# Odoo Billing Platform Architecture

## Summary

The Odoo billing platform is Telcobright's ISP/telecom billing and infrastructure management system. It combines **Odoo 17 CE** (master data, accounting, infrastructure management) with **Kill Bill 0.24.16** (subscription lifecycle, recurring invoicing, payments, dunning), connected through an **Apache Kafka** event bus for eventual consistency. A **Spring Boot API gateway** unifies access for a **React + MUI** frontend, with **Keycloak** providing JWT-based authentication.

The platform serves ISP/telecom clients (BTCL, etc.) with monthly recurring subscriptions for internet packages and telecom services (Hosted PBX, Voice Broadcast, Contact Center, Bulk SMS), along with infrastructure management (datacenter/server/container inventory) and artifact deployment pipelines.

**Repository**: `/home/mustafa/telcobright-projects/odoo/`

## Tech Stack

| Component | Technology | Version |
|-----------|-----------|---------|
| Backend ORM / Master Data | Odoo CE | 17.0 |
| Billing Engine | Kill Bill | 0.24.16 |
| API Gateway | Spring Boot | 3.4.3 |
| Frontend | React + MUI | React 19, MUI 7 |
| Build Tool (UI) | Vite | 8 |
| Authentication | Keycloak | 24.0.5 |
| Secrets Management | OpenBao/Vault | 2.1.0 |
| Event Bus (planned) | Apache Kafka | KRaft mode |
| Odoo Database | PostgreSQL 16 | Port 5433 |
| Kill Bill Database | PostgreSQL 16 | Port 5433 (was MySQL, migrated) |
| Document Storage DB | PostgreSQL 16 | Port 5433, database `odoo_documents` |
| Java (API Gateway) | JDK 21 | |
| Java (Kill Bill) | JDK 17 | Kill Bill requires JDK 11-17 |
| Python (Odoo) | Python 3.12 | |
| SSH Library | paramiko | 4.0.0 |

## Service Map and Ports

```
React UI (:5180)
    | JWT (Keycloak)
    v
Spring Boot API (:8180)
    |-- Odoo (:7169)         -- billing data, infra, artifacts (XML-RPC)
    |-- Kill Bill (:18080)   -- subscriptions, invoices, payments (REST)
    +-- Vault (:8200)        -- SSH keys (via Odoo)

Keycloak (:7104)             -- user authentication
PostgreSQL (:5433)           -- Odoo DB (odoo_billing), KB DB (killbill), Doc DB (odoo_documents)
```

| Port | Service | Protocol |
|------|---------|----------|
| 5180 | React UI (Vite dev) | HTTP |
| 7104 | Keycloak | HTTP |
| 7169 | Odoo | HTTP/JSON-RPC/XML-RPC |
| 8180 | Spring Boot API Gateway | HTTP/REST |
| 8200 | Vault/OpenBao | HTTP |
| 8900 | KB Webhook Relay (planned) | HTTP |
| 18080 | Kill Bill | HTTP/REST |
| 18000 | Kill Bill Debug | TCP (with `--debug` flag) |
| 5433 | PostgreSQL | TCP |

## Directory Structure

```
odoo/                                    # Repository root
|-- api/                                 # Spring Boot API gateway (Java 21)
|   |-- pom.xml                          # Spring Boot 3.4.3, groupId=com.telcobright
|   +-- src/main/java/com/telcobright/api/
|       |-- PlatformApiApplication.java
|       |-- controller/
|       |   |-- OdooProxyController.java    # Generic /{model}/{method} proxy
|       |   +-- KillBillProxyController.java # Pass-through /** proxy
|       |-- odoo/
|       |   +-- OdooClient.java             # XML-RPC client, auth caching
|       +-- config/
|           |-- OdooProperties.java
|           |-- KillBillProperties.java
|           +-- SecurityConfig.java          # JWT validation, CORS
|
|-- odoo-backend/                        # Odoo 17 server
|   |-- odoo-src/                        # Odoo 17 source (gitignored heavy)
|   |-- custom-addons/
|   |   |-- kb_integration/              # Kill Bill billing module (14 models)
|   |   |   |-- models/
|   |   |   |   |-- res_company.py       # KB tenant fields (api_key, api_secret, tenant_id)
|   |   |   |   |-- res_partner.py       # KB account fields, verification status
|   |   |   |   |-- product_template.py  # KB product name, category
|   |   |   |   |-- product_product.py   # KB plan name, billing period, trial
|   |   |   |   |-- account_move.py      # KB invoice ID
|   |   |   |   |-- account_payment.py   # KB payment ID
|   |   |   |   |-- sale_order_line.py   # KB subscription ID
|   |   |   |   |-- kb_sync_log.py       # Sync operation tracking (PG NOTIFY)
|   |   |   |   |-- product_rate_history.py  # Pricing history with effective dates
|   |   |   |   |-- product_tax_rate.py  # Tax rate history (VAT/AIT) with gazette refs
|   |   |   |   +-- doc_document.py      # Document mgmt (metadata in Odoo, binary in separate DB)
|   |   |   |-- views/                   # 11 XML view files
|   |   |   |-- security/ir.model.access.csv
|   |   |   +-- data/ir_sequence_data.xml
|   |   |-- infra_management/            # Infrastructure module (16 models)
|   |   |   |-- models/                  # Region, AZ, DC, Compute, Container, Network, SSH, Vault
|   |   |   |-- views/                   # 9 XML view files
|   |   |   |-- seed/                    # Device attributes + MikroTik models
|   |   |   +-- demo/                    # Sample data loader
|   |   +-- artifact_management/         # Artifact deployment module (7 models)
|   |       |-- models/                  # Project, Version, Deployment, Pipeline, Step, Template
|   |       |-- views/
|   |       +-- seed/routesphere_template.xml  # RouteSphere JAR deploy template
|   |-- odoo.conf                        # Server config (port 7169, PG 5433)
|   |-- start-odoo.sh
|   +-- venv/                            # Python virtualenv
|
|-- killbill/                            # Kill Bill server
|   |-- killbill-server/                 # KB source (cloned v0.24.16)
|   |-- catalogs/
|   |   |-- isp-catalog.xml             # 8 products, 8 plans (BDT, MONTHLY, EVERGREEN)
|   |   +-- overdue-config.xml          # WARNING 7d -> SUSPENDED 14d -> DISCONNECTED 30d
|   |-- killbill-server.properties       # PG 5433, multitenant=true, port 18080
|   |-- build.sh / start.sh / reset-db.sh / setup-tenant.sh / test-api.sh
|   +-- CLAUDE.md
|
|-- ui/                                  # React frontend
|   |-- src/
|   |   |-- services/                    # API clients (odoo.js, killbill.js, infra.js, artifacts.js, keycloak.js)
|   |   |-- pages/                       # 20+ page components
|   |   |   |-- Dashboard.jsx, Customers.jsx, CustomerDetail.jsx
|   |   |   |-- Subscriptions.jsx, Invoices.jsx, Payments.jsx
|   |   |   |-- Catalog.jsx, Products.jsx, ProductDetail.jsx
|   |   |   |-- Pricing.jsx, RateHistory.jsx, ARReport.jsx
|   |   |   |-- Settings.jsx, Tenants.jsx, Login.jsx
|   |   |   |-- infra/ (InfraMain, InfraTree, InfraDetailPane, InfraSSH, InfraDeviceCatalog)
|   |   |   +-- artifacts/ (ArtifactsMain, PipelineViewer)
|   |   |-- context/                     # AuthContext (dual KC/legacy), ThemeContext
|   |   |-- components/                  # PaymentReceipt, StatCard, StatusChip, TenantGuard
|   |   |-- layouts/                     # MainLayout, Sidebar, TopBar
|   |   +-- theme/                       # Themes (btcl=green, telcobright=blue)
|   |-- vite.config.js                   # Port 5180, proxy /api -> :8180
|   +-- tests/smoke.spec.js             # Playwright smoke test
|
|-- db/                                  # Database setup
|   |-- setup_databases.sh              # Creates killbill + odoo_documents DBs
|   |-- killbill-pg-schema.sql          # Kill Bill DDL (PG port)
|   |-- killbill-pg-compat.sql          # PG compatibility layer for KB
|   |-- odoo-documents-schema.sql       # doc_binary table for document storage
|   |-- pg_minimal.conf / pg_tuning.conf
|
|-- scripts/                             # Utility scripts (all use XML-RPC to Odoo)
|   |-- setup_products.py               # Product catalog: categories, attributes, variants, pricing
|   |-- setup_pricing.py                # Fix variant prices, create pricelists, VAT
|   |-- setup_gl_taxes.py              # GL accounts, journals, tax records, tax rate history
|   |-- setup_tax_categories.py         # BD VAT/tax classification (ITES, Software, etc.)
|   |-- populate_rate_history.py        # Historical pricing data
|   |-- e2e_first_subscription.py       # End-to-end: tenant mapping -> client -> subscription -> sync
|   +-- sync_invoices_payment.py        # Sync KB invoices to Odoo + record payments
|
+-- docs/                               # 21 documentation files across 8 directories
```

## Key Components

### 1. Odoo 17 CE (the "Brain")

Masters partners, product catalog, accounting/GL, tax, sales pipeline. Three custom modules:

**kb_integration** (14 models): Kill Bill integration with PG NOTIFY-based sync queue.
- Extends `res.company` with KB tenant credentials (`x_kb_api_key`, `x_kb_api_secret`, `x_kb_tenant_id`)
- Extends `res.partner` with KB account fields (`x_external_key`, `x_kb_account_id`, `x_verification_status`)
- Extends `product.template`/`product.product` with KB catalog mapping fields
- Extends `account.move`/`account.payment` with KB UUID idempotency keys
- New model `kb.sync.log` -- tracks every sync operation, uses PG NOTIFY channel `kb_sync`
- New model `product.rate.history` -- pricing history with effective dates and pricelist tiers
- New model `product.tax.rate` -- tax rate history per category/product with gazette references
- New models `doc.document` + `doc.mapping` -- document management with binary storage in separate DB (`odoo_documents`)

**infra_management** (16 models): Datacenter/server/container/network inventory with SSH operations.
- Hierarchy: Partner -> Region -> AZ -> Datacenter -> Resource Pool -> Compute -> Container
- Network devices, storage, IP addresses (polymorphic)
- Device catalog (attributes + models, seeded with MikroTik data)
- SSH key management with Vault integration (generate, deploy, verify, execute commands)
- BTCL server inventory imported from routesphere ssh-automation

**artifact_management** (7 models): Software registry and deployment pipelines.
- Project -> Version -> Deployment -> Pipeline -> Steps
- Deploy templates with variable substitution
- Includes "RouteSphere JAR Deploy" template (9 steps: stop, backup, upload, install, version, start, verify)
- Background SSH execution via paramiko with polling-based progress UI

### 2. Kill Bill 0.24.16 (the "Engine")

Executes subscriptions, recurring invoicing, payments, dunning. Multi-tenant via row-level isolation.

**Current Catalog** (8 products, 8 plans, all MONTHLY/EVERGREEN/IN_ADVANCE, BDT):

| Product | Plan Name | Price (BDT/mo) | Category |
|---------|-----------|-----------------|----------|
| Internet-100Mbps | internet-100mbps-monthly | 1,200 | BASE |
| Internet-50Mbps | internet-50mbps-monthly | 800 | BASE |
| Internet-200Mbps | internet-200mbps-monthly | 1,800 | BASE |
| HostedPBX | hosted-pbx-monthly | 1,200 | BASE |
| VoiceBroadcast | voice-broadcast-monthly | 1,800 | BASE |
| ContactCenter | contact-center-monthly | 850 | BASE |
| BulkSMS | bulk-sms-monthly | 500 | BASE |
| StaticIP | static-ip-monthly | 300 | ADD_ON |

**Overdue/Dunning**: WARNING (7d) -> SUSPENDED (14d) -> DISCONNECTED (30d). Auto-clears on payment.

**First Tenant**: API key `telcobright-isp`, secret `telcobright-isp-secret`, auth `admin:password`.

### 3. Spring Boot API Gateway (Port 8180)

Single entry point for React frontend. Generic proxy -- no per-model Java changes needed.

- `POST /api/odoo/{model}/{method}` -- Generic Odoo XML-RPC proxy (body: `{args, kwargs}`)
- `{GET|POST|PUT|DELETE} /api/kb/{path}` -- Kill Bill pass-through proxy (injects KB basic auth server-side)
- `GET /api/odoo/health` -- Public health check
- JWT validation via Keycloak (`spring-boot-starter-oauth2-resource-server`)
- CORS: allows `:5180` and `:8180`

Config: `/home/mustafa/telcobright-projects/odoo/api/src/main/resources/application.yml`

### 4. React Frontend (Port 5180)

MUI 7 + React 19 + Vite 8. Dual auth (Keycloak default, legacy fallback). Two themes (btcl/green, telcobright/blue).

**Routes**:
| Route | Component | Domain |
|-------|-----------|--------|
| `/` | Dashboard | Billing overview |
| `/customers`, `/customers/:id` | Customers, CustomerDetail | KB accounts |
| `/subscriptions` | Subscriptions | KB subscriptions |
| `/invoices` | Invoices | KB invoices + pay |
| `/payments` | Payments | Payment history + receipts |
| `/catalog` | Catalog | KB plans |
| `/products`, `/products/:id` | Products, ProductDetail | Odoo product catalog |
| `/pricing` | Pricing | Rate management |
| `/rate-history` | RateHistory | Historical pricing |
| `/reports/ar` | ARReport | Accounts receivable |
| `/settings` | Settings | Configuration |
| `/tenants` | Tenants | Super admin: multi-tenant |
| `/infra` | InfraMain | Infrastructure tree |
| `/infra/catalog` | InfraDeviceCatalog | Device models |
| `/infra/ssh` | InfraSSH | SSH keys + credentials |
| `/artifacts` | ArtifactsMain | Artifact deploy |

### 5. Keycloak (Port 7104)

Realm: `telcobright`. Client: `platform-ui` (public, PKCE flow).
Roles: `super_admin`, `tenant_admin`, `operator`, `readonly`.
JWT auto-refreshed every 10s by React. Spring Boot validates against KC public key.

### 6. Kafka Event Bus (Planned -- NOT YET IMPLEMENTED)

Designed but not built. Will use 8 topics for bidirectional sync:

**Odoo -> KB**: `odoo.partner.verified`, `odoo.catalog.changed`, `odoo.subscription.requested`, `odoo.payment.recorded`
**KB -> Odoo**: `kb.invoice.created`, `kb.payment.succeeded`, `kb.subscription.changed`, `kb.overdue.changed`

Partition key: `{tenant_api_key}:{external_key}` for per-customer ordering.

Planned services at `/home/mustafa/telcobright-projects/odoo-kb-sync/`:
- `kb-consumer/` -- reads odoo.* topics, calls KB REST API (Python)
- `odoo-consumer/` -- reads kb.* topics, calls Odoo XML-RPC (Python)
- `kb-webhook-relay/` -- FastAPI on port 8900, receives KB webhooks, publishes to kb.* topics

**Current sync approach**: PG NOTIFY on `kb_sync_log` table inserts, plus manual Python scripts (`e2e_first_subscription.py`, `sync_invoices_payment.py`).

## Database Architecture

Three PostgreSQL databases on port 5433:

| Database | Purpose | Owner |
|----------|---------|-------|
| `odoo_billing` | Odoo main (auto-created by Odoo) | mustafa |
| `killbill` | Kill Bill (PG-ported schema, row-level tenant isolation) | mustafa |
| `odoo_documents` | Binary document storage (separate to keep main DB fast) | mustafa |

**Document storage**: Metadata in `doc.document` (Odoo main DB), binary content in `doc_binary` table (odoo_documents DB), linked by UUID `storage_ref`. SHA256 checksum for deduplication.

**Kill Bill DB**: Originally MySQL, ported to PostgreSQL using compatibility layer (`killbill-pg-compat.sql`). JDBC URL: `jdbc:postgresql://127.0.0.1:5433/killbill`.

## Multi-Tenancy Design

Odoo multi-company (single DB, logical isolation) maps to Kill Bill row-level tenant isolation.

Each KB tenant = one Odoo `res.company`:
- `x_kb_api_key` on company -> `X-Killbill-ApiKey` header
- `x_kb_api_secret` on company -> `X-Killbill-ApiSecret` header

Products are shared across companies. Partners, invoices, payments, journals, taxes are scoped per company via Odoo's built-in `ir.rule` access rules.

Current tenant: `telcobright-isp` mapped to default company (id=1).

## Product Catalog in Odoo

Created via `scripts/setup_products.py` (XML-RPC, idempotent):

**Categories**: Internet Services > Bandwidth Plans / Dedicated Internet, SMS Services, Voice Services, Value Added Services

**Attributes**:
1. Bandwidth (radio): 10, 25, 50, 100, 200, 500 Mbps, 1 Gbps
2. Billing Cycle (select): Monthly, Quarterly, Yearly
3. SMS Package (radio): 10K, 50K, 100K, 500K, 1M SMS

**Products with Variants**: Shared Internet (21 variants), DIA (21 variants), Bulk SMS (5 variants)

**Simple Products**: IPLC, IP Transit, Colocation, Domain & Hosting, VoIP, MPLS VPN

**Pricing** (BDT, monthly): Shared Internet 10Mbps=3,000 ... 1Gbps=100,000. DIA ~2.5x shared. Bulk SMS 10K=4,000 ... 1M=250,000.

**Tax Categories** (per BD NBR classification): ITES (internet, SMS), Software, Computer Hardware, General Services, General Hardware. VAT rates: 15% standard, 7.5% ITES, 0% exempt. AIT 10% deduction at source.

## External Connections

### Connection to Routesphere

**There is no direct data integration between routesphere and the Odoo billing platform.** Specifically:
- Routesphere does NOT push CDRs to the billing system
- Billing does NOT pull from routesphere's database
- No Kafka topics or REST endpoints connect the two systems for data flow

The only connections are operational/infrastructure:
1. **Artifact deployment**: The `artifact_management` module has a "RouteSphere JAR Deploy" template (`odoo-backend/custom-addons/artifact_management/seed/routesphere_template.xml`) for deploying routesphere-core JARs to remote servers via SSH
2. **Infrastructure inventory**: BTCL server inventory (SBC containers, Kafka cluster, SMS servers) was imported from routesphere's `ssh-automation` directory into the `infra_management` module
3. **Vault sharing**: The platform overview references starting Vault from routesphere's vault directory (`/home/mustafa/telcobright-projects/routesphere/vault/start.sh`)
4. **Future billing for telecom services**: Kill Bill catalog includes HostedPBX, VoiceBroadcast, ContactCenter, BulkSMS -- services that routesphere delivers. The billing relationship is at the subscription level (monthly flat-rate), not usage-based CDR billing

**Future work** (not implemented): Usage-based billing for SMS/voice is listed as planned. If implemented, this would likely require routesphere CDR data to flow into Kill Bill for metered billing. No design exists yet for this integration.

### Kill Bill Integration (via Kafka -- planned, via scripts -- current)

Currently: Manual Python scripts perform sync operations (e2e_first_subscription.py, sync_invoices_payment.py).

Planned: Full Kafka event bus with bidirectional sync (see Kafka Event Bus section above).

### BTCL SMS Portal

Referenced in `killbill/CLAUDE.md`: The BTCL SMS Portal at `/home/mustafa/telcobright-projects/btcl-sms-portal` will push purchases to Kill Bill via REST API or Kafka topics. Flow: Portal purchase -> KB subscription -> Invoice -> Payment -> Activation event -> Portal callback.

### SSLCommerz Payment Gateway

Planned integration for online payments. Pattern: Frontend -> gateway redirect -> gateway callback -> KB payment recording -> overdue auto-clear.

## Data Flow

### Current (Manual Scripts)

```
1. setup_products.py  -> Odoo XML-RPC -> Creates product catalog
2. setup_gl_taxes.py  -> Odoo XML-RPC -> Creates GL accounts, journals, taxes
3. e2e_first_subscription.py:
   Odoo XML-RPC: create partner -> set KB credentials on company
   KB REST API:  create account -> create subscription
   Sync back:    write KB IDs to Odoo partner/SO
4. sync_invoices_payment.py:
   KB REST API:  fetch unpaid invoices
   Odoo XML-RPC: create account.move with x_kb_invoice_id
   KB REST API:  record payment
   Odoo XML-RPC: create account.payment with x_kb_payment_id
```

### Planned (Kafka Event-Driven)

```
Odoo -> Kafka -> KB:
  Partner verified -> odoo.partner.verified -> KB Consumer -> POST /accounts -> write accountId back
  Product changed  -> odoo.catalog.changed  -> KB Consumer -> POST /catalog/xml
  SO confirmed     -> odoo.subscription.requested -> KB Consumer -> POST /subscriptions
  Payment recorded -> odoo.payment.recorded -> KB Consumer -> POST /payments

KB -> Kafka -> Odoo:
  Invoice generated -> KB webhook -> Webhook Relay -> kb.invoice.created -> Odoo Consumer -> create account.move
  Payment succeeded -> KB webhook -> Webhook Relay -> kb.payment.succeeded -> Odoo Consumer -> create account.payment
  Subscription changed -> KB webhook -> Webhook Relay -> kb.subscription.changed -> update partner tags
  Overdue changed -> KB webhook -> Webhook Relay -> kb.overdue.changed -> update partner tags
```

### API Call Flow (Frontend)

```
React (:5180) --[JWT]--> Spring Boot (:8180) --[XML-RPC]--> Odoo (:7169)
React (:5180) --[JWT]--> Spring Boot (:8180) --[REST/Basic]--> Kill Bill (:18080)
Keycloak (:7104) provides JWT tokens via PKCE flow
```

## Configuration

### Odoo

File: `/home/mustafa/telcobright-projects/odoo/odoo-backend/odoo.conf`
- `http_port = 7169`
- `db_port = 5433`, `db_user = mustafa`
- `addons_path` includes `custom-addons/`
- Admin: `admin`/`admin`, DB: `odoo_billing`

### Kill Bill

File: `/home/mustafa/telcobright-projects/odoo/killbill/killbill-server.properties`
- Port 18080, PG `jdbc:postgresql://127.0.0.1:5433/killbill`
- `org.killbill.server.multitenant=true`
- Payment retry: 1, 3, 7 days
- Invoice safety: max 200 items/day
- Notification queue: STICKY_POLLING, 500ms sleep
- JAVA_HOME: `/home/mustafa/.sdkman/candidates/java/17.0.16-librca`

### Spring Boot API

File: `/home/mustafa/telcobright-projects/odoo/api/src/main/resources/application.yml`
- `server.port: 8180`
- Odoo: `http://127.0.0.1:7169`, db `odoo_billing`, user `admin`/`admin`
- Kill Bill: `http://127.0.0.1:18080`, user `admin`/`password`
- JWT issuer: `http://localhost:7104/realms/telcobright`

### React UI

File: `/home/mustafa/telcobright-projects/odoo/ui/vite.config.js`
- Port 5180
- Proxy: `/api` -> `http://127.0.0.1:8180`

### Document Storage

Hardcoded DSN in `doc_document.py`: `host=127.0.0.1 port=5433 dbname=odoo_documents user=mustafa password=mustafa`

## Deployment

### Local Development Quick Start

```bash
# 1. Odoo
cd odoo-backend && ./start-odoo.sh &

# 2. Spring Boot API
cd api && java -jar target/platform-api-1.0-SNAPSHOT.jar &

# 3. React UI
cd ui && npx vite --port 5180 &

# 4. Keycloak (optional)
export KEYCLOAK_ADMIN=admin KEYCLOAK_ADMIN_PASSWORD=admin
/opt/keycloak/bin/kc.sh start-dev --http-port=7104 &

# 5. Kill Bill (optional)
cd killbill && ./start.sh &

# 6. Setup tenant + catalog
cd killbill && ./setup-tenant.sh
```

### Kill Bill Build

```bash
cd killbill
./build.sh          # Full build (~5 min)
./build.sh --quick  # Changed modules only
./start.sh          # Start on :18080
./start.sh --debug  # Debug on :18000
./reset-db.sh       # Drop and recreate DB
./setup-tenant.sh   # Create tenant + upload catalog + overdue config
```

### Database Setup

```bash
cd db && ./setup_databases.sh
# Creates: killbill (PG), odoo_documents DBs
# odoo_billing is auto-created by Odoo on first run
```

### Seed Data Scripts

```bash
cd scripts
python setup_products.py          # Product catalog
python setup_pricing.py           # Variant pricing, VAT, pricelists
python setup_gl_taxes.py          # GL accounts, journals, tax records
python setup_tax_categories.py    # BD VAT classification
python populate_rate_history.py   # Historical pricing
python e2e_first_subscription.py  # Full E2E: tenant + client + subscription + sync
```

## Implementation Status

### Complete

- Kill Bill 0.24.16 installed, built, running with PostgreSQL backend
- Multi-tenant enabled, first tenant created with catalog and overdue config
- React UI with billing features (customers, subscriptions, invoices, payments, receipts, AR report)
- Odoo 17 CE running with 3 custom modules (kb_integration, infra_management, artifact_management)
- Product catalog (47 variants across shared internet, DIA, bulk SMS, plus 6 simple services)
- Spring Boot API gateway with generic Odoo/KB proxies
- Keycloak authentication with PKCE flow
- Infrastructure management with SSH operations and Vault integration
- Artifact deployment with RouteSphere JAR deploy template
- Document management with separate binary storage DB
- Tax rate and pricing history with effective dates

### Partially Complete

- Multi-company setup (single default company only, no additional tenants)
- Payment journals (bKash, Nagad, Rocket, etc.) -- may need configuration
- Accounting config (fiscal year, payment terms) -- scripts exist but may not be applied

### Not Started

- Kafka event bus setup (KRaft single-node for dev)
- Kafka consumer services (kb-consumer, odoo-consumer, kb-webhook-relay)
- Full bidirectional Kafka-based sync (partner, catalog, subscription, invoice, payment)
- SSLCommerz payment gateway integration
- Usage-based billing for SMS/voice (would require routesphere CDR integration)
- Email notification consumer
- Per-tenant invoice templates

## Integration Points with Routesphere

| Integration | Type | Status | Detail |
|-------------|------|--------|--------|
| RouteSphere JAR deployment | Operational (SSH pipeline) | Implemented | `artifact_management` module deploys routesphere-core JARs to remote servers |
| BTCL server inventory | Operational (data import) | Imported | 12 servers imported from routesphere ssh-automation into infra_management |
| Vault instance | Shared service | Active | Both projects use the same Vault at :8200 for SSH key storage |
| Billing for telecom services | Subscription (flat rate) | Designed | KB catalog has HostedPBX, VoiceBroadcast, BulkSMS plans but no CDR/usage metering |
| CDR-based usage billing | Data pipeline | Not designed | No existing integration for routesphere CDRs flowing into billing |
