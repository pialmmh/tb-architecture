# Telcobright Ecosystem — Architecture Overview

## What is Telcobright?

Telcobright is a **multi-tenant telecom platform** serving Bangladesh operators. It provides SMS delivery (A2P, P2P, OTP, promotional), voice call routing, voice broadcasting (VBS), omnichannel contact center (IM/CRM), and billing — all orchestrated through a set of interconnected services.

---

## Ecosystem Diagram

```
                                    ┌──────────────────────────┐
                                    │   SOFTSWITCH_DASHBOARD   │
                                    │   React 18 + Vite SPA    │
                                    │   (port 3000, PM2)       │
                                    │   SMS portal, VBS, CDR,  │
                                    │   user/partner mgmt      │
                                    └────────────┬─────────────┘
                                                 │ REST (JWT)
                                                 ▼
                              ┌──────────────────────────────────────┐
                              │          RTC-Manager                 │
                              │     Backend Microservices            │
                              │                                      │
                              │  ┌─────────────┐ ┌────────────────┐ │
                              │  │ API-Gateway  │ │  Security      │ │
                              │  │ (8001)       │ │  (8080, JWT)   │ │
                              │  └──────┬───────┘ └────────────────┘ │
                              │         │                             │
                              │  ┌──────┴───────────────────────┐    │
                              │  │ FreeSwitchREST (5071)        │    │
                              │  │ smsrest (6080) → gRPC Router │    │
                              │  │ smsprocessor (6071)          │    │
                              │  │ erp (8076)                   │    │
                              │  │ PaymentGateWay (8081)        │    │
                              │  │ WebSocket (8079)             │    │
                              │  │ csv_to_mysql (8808)          │    │
                              │  └──────────────────────────────┘    │
                              │                                      │
                              │  ┌──────────────┐ ┌───────────────┐  │
                              │  │ Eureka (8761)│ │ event-agg     │  │
                              │  │ discovery    │ │ (Quarkus,CDR) │  │
                              │  └──────────────┘ └───────────────┘  │
                              └──────────────────┬───────────────────┘
                                                 │
                         ┌───────────────────────┼───────────────────────┐
                         │                       │                       │
              ┌──────────▼──────────┐ ┌──────────▼──────────┐ ┌─────────▼─────────┐
              │   config-manager    │ │  routesphere-core    │ │     Sigtran       │
              │   (Spring Boot)     │ │    (Quarkus)         │ │   SS7 Gateway     │
              │   port 7071         │ │  port 19999 (V1 API) │ │  ports 8282-8285  │
              │   private IP only   │ │  port 18091 (V2 API) │ │  4 JVM instances  │
              │                     │ │                      │ │  (borak/khawaja)  │
              │  Tenant config tree │ │  SMS + Voice + IM    │ │                   │
              │  Debezium CDC       │ │  State machines      │ │  SRI + MT-FSM     │
              │  Nacos proxy        │ │  Billing/Rating      │ │  SCTP → operators │
              └─────────┬──────────┘ └──────────┬───────────┘ └─────────┬─────────┘
                        │                       │                       │
                        │ REST                  │ UDP (primary)         │
                        │◄──────────────────────┤ HTTP (fallback)       │
                        │                       ├──────────────────────►│
                        │                       │                       │
                        │                       │ Redis pub/sub         │
                        │                       │◄──────────────────────┤
                        │                       │ (SRI/MT responses)    │
                        │                       │                       │
              ┌─────────▼───────────────────────▼───────────────────────▼─────────┐
              │                     Shared Infrastructure                          │
              │                                                                    │
              │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌───────────────────┐  │
              │  │  MySQL   │  │  Kafka   │  │  Redis   │  │   FreeSWITCH     │  │
              │  │ per-     │  │  3-node  │  │ Sentinel │  │   (ESL, voice)   │  │
              │  │ tenant   │  │  KRaft   │  │ cluster  │  │                   │  │
              │  │ schemas  │  │          │  │          │  │                   │  │
              │  └──────────┘  └──────────┘  └──────────┘  └───────────────────┘  │
              └──────────────────────────────────────────────────────────────────────┘

              ┌──────────────────────┐      ┌──────────────────────────┐
              │   Night Watcher      │      │   Odoo + KillBill        │
              │   (Go + Python +     │      │   Billing System         │
              │    React dashboard)  │      │                          │
              │   port 7101 (status) │      │  Odoo 17 CE (port 8069) │
              │                      │      │  KillBill (port 8180)   │
              │   WAF/SIEM/HA/IPS    │      │  React UI (port 5180)   │
              │   Monitors all above │      │  PostgreSQL (port 5433) │
              │   services           │      │  Keycloak auth          │
              └──────────────────────┘      └──────────────────────────┘
```

---

## End-to-End Data Flows

### SMS Flow (A2P Transactional)

```
User Portal (SOFTSWITCH_DASHBOARD)
  │ POST /sms/send
  ▼
API-Gateway (RTC-Manager, port 8001) ──JWT verify──► Security (8080)
  │ routes to smsrest (6080) or direct to routesphere V2
  ▼
routesphere-core V2 API (port 18091)
  │ TransSmsPipeline: validate → partner lookup → rate → balance reserve
  │ Publish to Kafka topic: TransSmsRest-<tenant>
  ▼
TransSmsRestConsumer (routesphere-core, per-tenant)
  │ processAsync → DB insert → Kafka OmniQueue
  ▼
SmsQueueConsumer (TPS rate-limited)
  │ SoftSwitch → SmscBean → SigTranSmsRegistry
  ▼
SmsMachineFactory state machine
  │ INIT → SRI (UDP to Sigtran) → MT-FSM (UDP to Sigtran)
  │ Responses via Redis pub/sub
  ▼
Sigtran Gateway (ports 8282-8285)
  │ SCTP/M3UA → operator STP → mobile subscriber
  ▼
CDR generation → mdr database + CSV files
  │ Kafka topic → event-aggregator (RTC-Manager)
  ▼
Reports visible in SOFTSWITCH_DASHBOARD
```

### Voice Call Flow (ESL)

```
SIP INVITE → FreeSWITCH
  │ ESL CHANNEL_CREATE event
  ▼
EslCallRegistry → EslCallMachineFactory
  │ INIT: CallAdmission → Partner ID → Multi-level balance reserve
  │ TRYING → RINGING → ANSWERED
  │ Periodic billing every 60s
  ▼
HANGUP → COMPLETED/FAILED
  │ Balance settlement/refund
  │ CDR → CdrCsvWriter → CSV + Kafka
  ▼
event-aggregator (RTC-Manager) → partitioned MySQL storage
  │
Reports in SOFTSWITCH_DASHBOARD + Odoo billing (future)
```

---

## Service Map

| Service | Framework | Port(s) | Server(s) | Purpose |
|---------|-----------|---------|-----------|---------|
| **routesphere-core** | Quarkus 3.x, Java 21 | 19999 (V1), 18091 (V2) | Per-tenant LXC | Core SMS/voice/IM engine |
| **config-manager** | Spring Boot 3.2, Java 21 | 7071 (private IP) | Per-tenant LXC | Tenant config, CDC, entity CRUD |
| **Eureka-Server** | Spring Boot | 8761 | Shared | Service discovery |
| **API-Gateway** | Spring WebFlux | 8001 | Per-deployment | JWT auth, routing, rate limiting |
| **Security** | Spring Boot | 8080 | Per-deployment | User/role/JWT management |
| **FreeSwitchREST** | Spring Boot | 5071 | Per-deployment | Voice API, DID, routes, CDR |
| **smsrest** | Spring Boot | 6080 | Per-deployment | SMS REST API + gRPC routing |
| **smsprocessor** | Spring Boot | 6071 | Per-deployment | SMS business logic |
| **erp** | Spring Boot | 8076 | Per-deployment | Billing/invoicing integration |
| **event-aggregator** | Quarkus | 9191 | Per-deployment | Outgoing CDR aggregation |
| **event-aggregator-incoming** | Quarkus | 9192 | Per-deployment | Incoming CDR aggregation |
| **Sigtran** | Restcomm jSS7, Java 8 | 8282-8285 (HTTP/UDP) | Dedicated server | SS7/GSM MAP gateway |
| **Night Watcher** | Go + Python + React | 7101 (status API) | Per-tenant LXC | WAF/SIEM/HA monitoring |
| **Odoo** | Odoo 17 CE (Python) | 8069 | Dedicated | ERP, invoicing, product catalog |
| **KillBill** | Kill Bill 0.24 (Java) | 8180 (via Spring Boot API) | Dedicated | Subscription billing |
| **SOFTSWITCH_DASHBOARD** | React 18 + Vite | 3000 (PM2 + serve) | Per-deployment | Web portal for all operations |

---

## Shared Resources

### MySQL
- **Per-tenant schemas**: `bdcom_sms`, `btcl_sms`, `l3ventures_sms`, `ccl_sms`, etc.
- **Reseller schemas**: `res_*` prefix, discovered dynamically by config-manager
- **MDR databases**: Per-tenant CDR storage with monthly partitioning
- **Odoo/KillBill**: Separate PostgreSQL (port 5433)

### Kafka (3-node KRaft cluster)
- **SMS queuing**: Per-tenant topics (`TransSmsRest-<tenant>`, OTP/Trans/Retry/Promo queues)
- **Config events**: `config_event_loader_<tenant>` (CDC changes)
- **CDR aggregation**: `PLAINTEXT` (outgoing), `PLAINTEXT2` (incoming)
- **SMS delivery reports**: Tenant-specific DLR topics

### Redis (Sentinel cluster)
- **SMS dedup locks**: `sms:lock:{uniqueId}` (20-min TTL)
- **Sigtran responses**: `sriresponse-<tenant>`, `mtfsmpublisher-<tenant>` pub/sub channels
- **Config fast-reload**: Per-tenant pub/sub channels
- **Session caching**: API Gateway and Security service

### WireGuard / BGP Overlay
- **Subnet scheme**: `10.10.x.0/24` per host (199, 198, ... decrementing)
- **FRR BGP**: Per-host router announces app subnets
- **VPN**: Developers connect via WireGuard/OpenVPN, get routes to `10.10.0.0/16`
- **config-manager** binds to `10.10.x.x` or `10.9.x.x` (VPN) — never `0.0.0.0`

---

## Tenants

| Tenant | DB Name | Services Enabled | Notes |
|--------|---------|-------------------|-------|
| `bdcom` | `bdcom_sms` | SMS | Production, CDC disabled |
| `btcl` | `btcl_sms` | SMS | Production, CDC disabled |
| `link3` | `l3ventures_sms` | SMS | Production, CDC disabled |
| `l3venture` | `l3ventures_sms` | SMS | Same DB as link3, different config |
| `ccl` | `ccl_sms` | SMS | Production |
| `brilliant` | `brilliant_sms` | SMS | Production |
| `ks_network` | `ks_network_sms` | SMS | Production |
| `btcl_contact_center` | — | IM/CRM + VBS | Contact center |
| `btcl_pbx` | — | PBX | CDC enabled |
| `btcl_voicebroadcasting` | — | VBS | CDC enabled (Kafka only) |

---

## Project Documentation Index

| Project | Doc | Summary |
|---------|-----|---------|
| [Routesphere Core](../ROUTESPHERE_PROJECT_OVERVIEW.md) | Existing | Multi-tenant SMS/voice/IM routing engine (Quarkus) |
| [Config-Manager](config-manager.md) | New | Spring Boot tenant config, CDC, entity CRUD (port 7071) |
| [SOFTSWITCH_DASHBOARD](softswitch-dashboard.md) | New | React 18 SPA — SMS portal, VBS, CDR, user management |
| [RTC-Manager](rtc-manager.md) | New | 14+ Spring Boot/Quarkus microservices — API gateway, security, voice, SMS, CDR |
| [Sigtran](sigtran.md) | New | SS7/GSM MAP gateway — 4 JVM instances, SCTP to operators |
| [Night Watcher](night-watcher.md) | New | Go+Python+React monitoring — WAF/SIEM/HA/IPS per tenant |
| [Odoo + KillBill](odoo-billing.md) | New | Billing — Odoo 17 CE + Kill Bill 0.24, PostgreSQL, Keycloak |
| [BTCL SMS Portal](btcl-sms-portal.md) | New | Next.js 15 customer portal — purchase VBS, HCC, PBX, Bulk SMS (BTCL) |
| [GPM Architecture](../gpm-statemachine/GPM_OBJECTIVES_AND_REQUIREMENTS.md) | Existing | Generic Processing Machine — future refactoring plan |
| [GPM Implementation](../gpm-statemachine/GPM_IMPLEMENTATION_PLAN.md) | Existing | GPM state machine, services, phased rollout |
| [Trans SMS Pipeline](../trans-sms-async-pipeline.md) | Existing | Async Kafka pipeline for transactional SMS |

### BTCL PBX / Voice Deep-Dive Docs

| Doc | Summary |
|-----|---------|
| [Voice Features — Dashboard](voice-softswitch-features-dashboard.md) | 25 voice pages, CCL profile, WebRTC softphone, per-role menus |
| [Voice Features — RTC-Manager](voice-softswitch-features-rtcmanager.md) | 35 controllers (~280 endpoints), ESL event handling, call center ACD |
| [FusionPBX Integration](fusionpbx-integration.md) | 22 wrapped features, dual PG+REST API, 14 entities, 26 controllers |
| [BTCL Service Topology](btcl-service-topology.md) | IP addresses, DB credentials, nginx routing, troubleshooting commands |
| [ESL vs SMS State Machine](../esl-vs-sms-statemachine-comparison.md) | Stability gap analysis — IDLE state, timer leaks, error handling |
| [BTCL PBX Call Pipeline](../btcl-pbx-call-pipeline-deep-dive.md) | End-to-end call flow, admission, dialplan, billing, CDR |

### L3venture SMS Deep-Dive Docs

| Doc | Summary |
|-----|---------|
| [Frontend API Inventory](l3venture-frontend-api-inventory.md) | 223+ API calls across 3 gateways |
| [Backend API Inventory](l3venture-backend-api-inventory.md) | 450+ endpoints across FreeSwitchREST, smsrest, Security |
| [UI Routes & Navigation](l3venture-ui-routes-and-navigation.md) | 70 routes, 8 roles, voice/SMS mode mapping |

---

## Authentication Flow

```
Browser → SOFTSWITCH_DASHBOARD (React)
  │ POST /AUTHENTICATION/user/authenticate (email + password)
  ▼
API-Gateway (8001) → Security Service (8080)
  │ Returns JWT token
  ▼
All subsequent requests carry: Authorization: Bearer <JWT>
  │
API-Gateway GlobalFilter validates JWT on every request
  │ Public endpoints bypass (login, health, version)
  ▼
Backend services trust the gateway-validated request
```

---

## Deployment Topology

Each tenant deployment typically includes:
1. **LXC container** with routesphere-core + config-manager
2. **Shared Kafka cluster** (3 KRaft nodes)
3. **Redis Sentinel** cluster (3 nodes)
4. **MySQL** (per-tenant schema)
5. **Sigtran gateway** (dedicated server, 4 instances)
6. **Night Watcher** (per-tenant LXC)
7. **RTC-Manager services** (API Gateway + dependent services)
8. **SOFTSWITCH_DASHBOARD** (static React build, PM2)

Deploy scripts use SSH inventory (`ssh-automation/servers/`) for passwordless operations. Git tags on parent repos (`vYYYY.MM.DD-HHMM`) track releases. Both routesphere-core and config-manager write `version.txt` on deploy, queryable via REST.
