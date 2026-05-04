# RTC-Manager -- Architecture Overview

## Summary

RTC-Manager is a backend microservices ecosystem built by Telcobright for unified communications management. It provides hosted IP PBX, voice broadcasting, A2P SMS, contact center, and billing/payment capabilities. The system manages FreeSWITCH-based voice infrastructure alongside an SMS aggregation platform, serving telecom operators (BTCL, Link3/L3Venture, CCL) through tenant-specific deployments. All inter-service routing passes through a Spring Cloud Gateway with JWT-based auth, and services discover each other via Eureka. Kafka is used for event streaming across SMS CDR aggregation, config change propagation, and payment workflows.

## Tech Stack

| Component           | Technology                                                      |
|---------------------|-----------------------------------------------------------------|
| Language            | Java 21                                                         |
| Build               | Maven                                                           |
| Web frameworks      | Spring Boot 3.2.5 -- 3.5.3 (most services), Quarkus 3.19 -- 3.24 (event-aggregator, Grpc-Router, sms-delivery-report) |
| API Gateway         | Spring Cloud Gateway (reactive, WebFlux)                        |
| Service discovery   | Netflix Eureka (Spring Cloud 2023.0.x)                          |
| Auth                | Spring Security + JWT (jjwt 0.11.5)                             |
| Databases           | MySQL (primary, via JPA/Hibernate + R2DBC in gateway), PostgreSQL (FusionPBX) |
| Messaging           | Apache Kafka (Spring Kafka 3.2.2, SmallRye Reactive Messaging)  |
| gRPC                | grpc-java 1.56 -- 1.64, Protobuf 3.23 -- 3.25                  |
| Stream processing   | Apache Flink 1.18.1                                             |
| In-process queue    | Chronicle Queue 5.27ea5                                         |
| Scheduling          | Spring Quartz                                                   |
| WebSocket           | Spring WebSocket + XMPP (Smack/Whack/Tinder)                   |
| PDF generation      | OpenPDF 1.3.35                                                  |
| Email               | Spring Mail                                                     |
| API docs            | SpringDoc OpenAPI 2.6.0 (Security, FreeSwitchREST, smsrest)    |
| Structured logging  | Logstash Logback Encoder 7.3                                    |
| Payment gateways    | SSLCommerz, Nagad                                               |
| FreeSWITCH ESL      | org.freeswitch.esl.client 0.9.2                                 |
| Redis               | Lettuce 6.6.0 (Apache-Flink module)                             |

## Directory Structure

```
RTC-Manager/
  API-Gateway/          -- Spring Cloud Gateway, JWT auth, request routing (port 8001)
  ConfigManager/        -- Centralized config + Kafka CDC + tenant registry (port 7071)
  Eureka-Server/        -- Netflix Eureka service discovery (port 8761)
  Security/             -- JWT auth service, user/role/permission CRUD (port 8080)
  FreeSwitchREST/       -- PBX management REST API: extensions, gateways, CDR, recordings (port 5071)
  FreeSwitchEsl/        -- FreeSWITCH ESL event handler + call state machine (port 5070)
  smsrest/              -- SMS management REST API: campaigns, rate plans, billing rules (port 6080)
  smsprocessor/         -- SMS Kafka producer + send endpoints (port 6071)
  sms-delivery-report/  -- Quarkus Kafka consumer for SMS delivery report processing
  event-aggregator/     -- Quarkus CDR aggregator: Kafka consumer -> DB/CSV (topic: PLAINTEXT)
  event-aggregator-incoming/ -- Quarkus CDR aggregator for incoming events (topic: PLAINTEXT2)
  Grpc-Router/          -- Quarkus gRPC server for SMS routing + WebSocket + Kafka CDC (ports 9000/9001)
  Apache-Flink/         -- Flink stream processing jobs: Kafka -> gRPC -> Redis
  Kafka/                -- Kafka campaign task dumper with Quartz scheduling
  erp/                  -- ERP/billing integration with Dolibarr (port 8076)
  PaymentGateWay/       -- SSLCommerz + Nagad payment processing (port 8081)
  csv_to_mysql/         -- CSV data import utility (port 8808)
  WebSocket-SpringBoot/ -- WebSocket server + XMPP bridge (port 8079)
  partitioned-repo/     -- Shared library: Hibernate entities for partitioned tables
  util/                 -- Shared utility library (build-only, no standalone port)
  xState/               -- State machine experiment (minimal, no dependencies)
  git_StateMachineLibrary_Test/ -- State machine library test project
  Tools/                -- Deployment shell scripts per tenant
  ToolsDeploy/          -- Legacy deployment instructions
```

## Services

### Eureka-Server
- **Framework:** Spring Boot 3.2.5, Spring Cloud Netflix Eureka Server
- **Port:** 8761
- **Key config:** `eureka.instance.prefer-ip-address=true`, standalone mode (no self-registration)
- **Role:** Central service registry. All Spring Boot services register here and discover peers by application name.
- **Source:** `/home/mustafa/telcobright-projects/RTC-Manager/Eureka-Server/`

### API-Gateway
- **Framework:** Spring Boot 3.2.5, Spring Cloud Gateway (reactive/WebFlux)
- **Port:** 8001
- **Database:** R2DBC MySQL (`tenant_master` DB for local profile; `btcl_sms` for btcl profile)
- **Discovery:** Eureka client (`spring.cloud.gateway.discovery.locator.enabled=true`, lowercase service IDs)
- **Auth:** GlobalFilter `HttpAuthPolicyMatcher` validates JWT tokens by calling Security service at `http://localhost:8080/auth/validateAccess?token=`
- **Routing:** Eureka-based auto-discovery routes requests to `/{SERVICE_NAME}/...` paths. Route targets use uppercase Eureka service names (e.g., `/AUTHENTICATION/auth/login`, `/FREESWITCHREST/api/v1/...`, `/SMSREST/campaign/...`).
- **Security policies:** Role-based policies -- `HttpAuthPolicySuperAdmin`, `ResellerPolicy`, `CallingPortalUserPolicy`, `BtrcUserPolicy`, `WebRtcPolicy`, `SmsSenderPolicy`, `ReadOnlyPolicy`, `DenyAllPolicy`
- **Public endpoints:** Extensive list in `PublicEndpointRegistry.java` -- auth, partner CRUD, OTP, PBX CRUD, payment, monitoring endpoints bypass JWT
- **Audit logging:** All non-public requests are audit-logged to MySQL via `AuditLogRepository`
- **R2DBC pool:** 10 initial / 50 max connections, 30s create timeout, 10m max lifetime, background eviction every 30s
- **Profiles:** `local`, `ccl76`, `link3`, `btcl`
- **Source:** `/home/mustafa/telcobright-projects/RTC-Manager/API-Gateway/`

### Security (Authentication Service)
- **Framework:** Spring Boot 3.2.5, Spring Security 6, JWT (jjwt 0.11.5)
- **Port:** 8080
- **Eureka name:** `AUTHENTICATION`
- **Databases:** MySQL (primary, tenant-specific per profile), PostgreSQL (FusionPBX at `103.95.96.100:5432/fusionpbx`)
- **Key endpoints:**
  - `POST /auth/login` -- JWT login
  - `POST /auth/webrtc/login` -- WebRTC login
  - `POST /auth/createUser` -- user registration
  - `POST /auth/validateAccess` -- token validation (called by API-Gateway)
  - `POST /auth/createRole`, `/getRoles`, `/deleteRole/{id}`, `/setRolePermissions/{roleId}` -- RBAC
  - `POST /auth/createPermission`, `/getPermissions`, `/editPermission` -- permission CRUD
  - `POST /permissions/available-menus`, `/get-user-permissions`, `/save`, `/create-pbx-user` -- PBX menu permissions
  - `POST /token/create-api-key`, `/delete-api-key`, `/get-active-api-keys-for-user` -- API key management
  - `POST /system-api/get-reseller-database-with-token`, `/add-reseller-database`, `/get-idpartner-from-jwt` -- internal service utilities
  - `POST /admin/create-admin-user`, `/manage-user-roles` -- admin operations
- **Inter-service secret:** `security.secret-header-value=oi-secur1ty-u-better=0bey-ur-Mast3r`
- **Connects to:** ConfigManager (`http://localhost:7071`)
- **Profiles:** `local`, `ccl76`, `link3`, `btcl`, `dev`
- **API docs:** SpringDoc OpenAPI at `/swagger-ui.html`
- **Source:** `/home/mustafa/telcobright-projects/RTC-Manager/Security/`

### ConfigManager
- **Framework:** Spring Boot 3.2.5
- **Port:** 7071
- **Eureka name:** `ConfigManager`
- **Databases:** MySQL (primary, tenant-specific), PostgreSQL (FusionPBX)
- **Kafka:** Producer + consumer, bootstrap at `localhost:9092`, consumer group `myGroup`
- **Key endpoints:**
  - `POST /get-tenant-root` -- get tenant configuration root
  - `POST /get-global-tenant-registry` -- list all tenant registrations
- **Role:** Centralized configuration management with Kafka-based CDC (Change Data Capture) for propagating config changes to downstream services.
- **Profiles:** `local`, `ccl76`, `link3`, `btcl`, `kafka-fix`
- **Source:** `/home/mustafa/telcobright-projects/RTC-Manager/ConfigManager/`

### FreeSwitchREST (PBX Management API)
- **Framework:** Spring Boot 3.2.5
- **Port:** 5071
- **Eureka name:** `FreeswitchRest`
- **Databases:** MySQL (primary), PostgreSQL (FusionPBX)
- **Dependencies:** Chronicle Queue 5.27ea5, Spring Kafka, Quartz scheduler, OpenPDF, Spring Mail, FreeSWITCH ESL client, shared `util` library
- **Key endpoints (selected):**
  - **PBX core:** `/api/v1/extensions/...`, `/api/v1/gateways/...`, `/api/v1/domains/...`
  - **Routing:** `/route/create-route`, `/route/get-routes`, `/route/delete-route`
  - **CDR/recordings:** `/api/v1/cdr/...`, `/api/recordings/...`, `/api/v1/recordings/...`
  - **Call center:** `/api/call-center/v1/agents/...`, `/api/call-center/v1/queues/...`, `/api/call-center/v1/tiers/...`
  - **Active calls:** `/api/v1/active-calls/list`, `/details`, `/hangup`, `/transfer`, `/hold`, `/mute`, `/conference`
  - **Call forward:** `/api/v1/call-forward/...`
  - **Ring groups:** `/api/v1/ring-groups/...`
  - **Conferences:** `/api/v1/conferences/...`
  - **IVR:** `/api/v1/fusionpbx-ivr/...`
  - **Contacts:** `/api/v1/fusionpbx-contact/...`
  - **Inbound/outbound routes:** `/api/v1/inbound-routes/...`, `/api/v1/outbound-routes/...`
  - **Destinations:** `/api/v1/destinations/...`
  - **Registrations:** `/api/v1/registrations/...`
  - **Partner:** `/partner/create-partner`, `/partner/get-partners`, `/partner/update-partner`
  - **OTP:** `/otp/send`, `/otp/varify`, `/otp/email/send`, `/otp/email/verify`
  - **Package/billing:** `/package/purchase-package`, `/package/topup`, `/package/setup-postpaid-credit`
  - **Digit filter rules:** `/api/digit-filter-rules/...`
  - **SIP monitoring:** `/sip-options-monitor/status`, `/configs`
  - **Broadcasting:** `/api/broadcast/play-audio`, `/api/call-broadcast/v1/...`
  - **Summary reports:** `/api/summry-reports/dom-in-iptsp`, `/dom-out-iptsp`
- **Connects to:** Security (`http://localhost:8080`), FreeSwitchEsl (`http://localhost:5070`), ConfigManager (`http://localhost:7071`), Eureka
- **SIP options monitoring:** Auto-discovers FreeSWITCH gateways for SIP OPTIONS health checks
- **Profiles:** `local`, `ccl76`, `link3`, `btcl`
- **API docs:** SpringDoc OpenAPI
- **Source:** `/home/mustafa/telcobright-projects/RTC-Manager/FreeSwitchREST/`

### FreeSwitchEsl (ESL Event Handler)
- **Framework:** Spring Boot 3.2.5
- **Port:** 5070
- **Eureka name:** `FreeSwitchEsl`
- **Databases:** MySQL (primary), PostgreSQL (FusionPBX)
- **Dependencies:** FreeSWITCH ESL client, Chronicle Queue, Kafka, gRPC (server + client), OkHttp, Netty, shared `util` library
- **Key endpoints:**
  - `POST /profile-path` -- get FreeSWITCH profile path
  - `POST /getConcurrentCall` -- current concurrent call count
  - `POST /get-partner-details` -- partner details lookup
  - `POST /get-btcl-calls`, `/get-outgoing-calls` -- call lists
  - `POST /get-partner-concurrent-call-limit`, `/concurrent-call-profile`, `/concurrent-call-profile-partner`, `/concurrent-call-partner-wise` -- concurrent call management
  - `POST /number` -- MNP (Mobile Number Portability) lookup
- **Cost/revenue config:** Configurable per-minute rates for incoming/outgoing calls and per-message SMS costs
- **gRPC:** Both server and client via protobuf (batch CRUD, prepaid accounting, WAL batch operations)
- **Profiles:** `local`, `ccl` (custom), `btcl`
- **Source:** `/home/mustafa/telcobright-projects/RTC-Manager/FreeSwitchEsl/`

### smsrest (SMS Management REST API)
- **Framework:** Spring Boot 3.2.5
- **Port:** 6080
- **Eureka name:** `smsrest`
- **Database:** MySQL (tenant-specific per profile)
- **Dependencies:** gRPC client, Spring Kafka, shared `util` library, OpenCSV, Apache POI, SpringDoc OpenAPI
- **gRPC:** Connects to Grpc-Router at `45.251.228.3:9000` for SMS delivery
- **Key endpoints (selected):**
  - **SMS sending:** `POST /sms/send`, `POST /api/v2/promo/sendSms`, `POST /api/v2/trans/sendSms`, `GET /api/v2/promo/sendSmsViaGet`
  - **Campaigns:** `/campaign/get-campaigns`, `/save-campaign`, `/disableCampaign`, `/enableCampaign`, `/get-campaign-statistics`
  - **Voice broadcast:** `/voice-broadcast/save-voice-broadcast`, `/get-voice-broadcasts`, `/download-audio/{campaignId}`
  - **Rate plans:** `/api/rate-plan/create`, `/getAll`, `/search`, `/update`, `/deleteRatePlan`
  - **Rate tasks:** `/api/ratetask/createRateTask`, `/insertDirect`, `/insertBulkDirect`
  - **Policies:** `/api/policies/create`, `/getAll`, `/update`, `/delete`
  - **Retry rules:** `/api/retryRules/create`, `/getAll`, `/getByPolicyId`
  - **Throttling:** `/api/throttlingRules/create`, `/getAll`, `/getByPolicyId`
  - **SMS queue:** `/api/sms-queue/create`, `/getAll`, `/update`, `/delete`
  - **Contacts:** `/contact-group/...`, `/contact-item/...`, `/uploadContactItems`
  - **DND:** `/dnd-group/...`, `/dnd-list/...`
  - **Forbidden words:** `/forbidden-group/...`, `/forbidden-word-list/...`
  - **DID numbers:** `/did-number/create`, `/create-from-excel`, `/get-all`, `/get-attributes`
  - **SMS templates:** `/approved-sms-template/create`, `/update`, `/get-all`, `/get-by-partner`
  - **Dashboard:** `/api/admin/dashboard/sms-and-campaign-summary`, `/get-top-5-partner-sms-count`, `/incoming/sms-stats`, `/outgoing/sms-stats`
  - **Billing rules:** `/api/jsonbillingrules/getAllRules`
- **Connects to:** Security (`http://localhost:8080`), Grpc-Router (gRPC at `45.251.228.3:9000`), Eureka
- **Profiles:** `ccl76`, `link3`, `btcl`, `dev`
- **Source:** `/home/mustafa/telcobright-projects/RTC-Manager/smsrest/`

### smsprocessor (SMS Kafka Producer)
- **Framework:** Spring Boot 3.2.5
- **Port:** 6071
- **Eureka name:** `smsprocessor`
- **Database:** MySQL (`ofbiz` DB)
- **Kafka:** Producer + consumer at `localhost:9092`
- **Key endpoints:**
  - `POST /sendsms/otp` -- send OTP SMS
  - `POST /sendsms/transactional` -- send transactional SMS
  - `POST /sendsms/bulk` -- send bulk SMS
  - `POST /send-sms` -- Kafka-routed SMS send
- **Source:** `/home/mustafa/telcobright-projects/RTC-Manager/smsprocessor/`

### event-aggregator (CDR Aggregator -- Outgoing)
- **Framework:** Quarkus 3.24.4 (SmallRye Reactive Messaging Kafka)
- **Port:** default Quarkus (8080 unless overridden)
- **Database:** MySQL (`btcl_sms`)
- **Kafka topic:** `PLAINTEXT` (consumer group: `quarkus-consumer-group-cdr`)
- **Dependencies:** `partitioned-repo` library (shared Hibernate entities)
- **Functionality:** Consumes SMS events from Kafka, writes CDR data to MySQL partitioned tables or CSV files. Configurable: `cdr.save.toDb=false`, `cdr.save.toCsv=true`. Partition management: 7-day span, 7-day retention.
- **Source:** `/home/mustafa/telcobright-projects/RTC-Manager/event-aggregator/`

### event-aggregator-incoming (CDR Aggregator -- Incoming)
- **Framework:** Quarkus 3.24.4 (SmallRye Reactive Messaging Kafka)
- **Port:** default Quarkus
- **Database:** MySQL (`btcl_sms`)
- **Kafka topic:** `PLAINTEXT2` (consumer group: `quarkus-consumer-group-cdr-incoming`)
- **Dependencies:** `partitioned-repo` library
- **Functionality:** Same as event-aggregator but for incoming SMS events on a separate Kafka topic.
- **Source:** `/home/mustafa/telcobright-projects/RTC-Manager/event-aggregator-incoming/`

### sms-delivery-report
- **Framework:** Quarkus 3.24.4 (SmallRye Reactive Messaging Kafka)
- **Port:** default Quarkus
- **Database:** MySQL (`ofbiz` at `119.40.81.52:3306`)
- **Kafka topic:** `PLAINTEXT` (consumer group: `quarkus-consumer-group`)
- **Functionality:** Consumes SMS events from Kafka and processes delivery reports. Thread pool: 20 core / 20 max threads.
- **Source:** `/home/mustafa/telcobright-projects/RTC-Manager/sms-delivery-report/`

### Grpc-Router (SMS gRPC Server)
- **Framework:** Quarkus 3.19.3 (gRPC, REST, WebSocket, Kafka, Hibernate/Panache)
- **HTTP port:** 9001 (bound to `45.251.228.3`)
- **gRPC port:** 9000 (bound to `45.251.228.3`)
- **Database:** MySQL (`telcobright` at `45.251.228.4:3306`)
- **Kafka:** Consumes `telcobright_all_tables` topic for config reload (CDC from ConfigManager)
- **REST client:** Connects to extensions API at `http://45.251.228.3:7070`
- **gRPC service:** `SmsService.SendSms(SmsRequest) returns (SmsResponse)` -- receives SMS tasks from smsrest, routes to downstream delivery
- **Proto definition:** `sms.proto` -- `SmsRequest` message with 40 fields covering full SMS lifecycle (phone numbers, campaign, retry state, encoding, etc.)
- **Source:** `/home/mustafa/telcobright-projects/RTC-Manager/Grpc-Router/`

### Apache-Flink (Stream Processing)
- **Framework:** Apache Flink 1.18.1 (standalone, not Spring Boot)
- **Main class:** `org.example.ApacheFlink`
- **Dependencies:** Flink Kafka connector 3.1.0, gRPC client, Lettuce Redis client 6.6.0, RocksDB state backend
- **Functionality:** Kafka-to-gRPC stream processing with Redis for state management. Uses Flink's RocksDB state backend for checkpointing.
- **Protobuf:** Same `sms.proto` as Grpc-Router
- **Source:** `/home/mustafa/telcobright-projects/RTC-Manager/Apache-Flink/`

### erp (ERP/Billing Integration)
- **Framework:** Spring Boot 3.3.4
- **Port:** 8076
- **Eureka name:** `ERP`
- **Database:** MySQL (`dolibarr` DB)
- **Kafka:** Producer + consumer at `localhost:9092`
- **Dependencies:** FreeSWITCH ESL client (for direct call data)
- **Functionality:** Integrates with Dolibarr ERP for invoicing and billing. No REST controllers found -- primarily Kafka-driven.
- **Source:** `/home/mustafa/telcobright-projects/RTC-Manager/erp/`

### PaymentGateWay
- **Framework:** Spring Boot 3.5.3
- **Port:** 8081
- **Database:** MySQL (tenant-specific per profile)
- **Kafka topics:** `service_request`, `service_approved`, `payment_approved`, `tenant_master`, `invoice`
- **Payment providers:** SSLCommerz (4 store configs: VBS, PBX, CC, SMS), Nagad
- **Key endpoints:**
  - `POST /api/payment/unified/purchase` -- unified purchase flow
  - `POST /api/payment/ssl/vbs/initiate`, `/pbx/initiate`, `/cc/initiate`, `/sms/initiate` -- SSLCommerz initiation per service
  - `POST /api/payment/ssl/vbs/validate`, `/pbx/validate`, `/cc/validate`, `/sms/validate` -- SSLCommerz validation
  - `POST /api/payment/ssl/vbs/ipn`, `/pbx/ipn`, `/cc/ipn`, `/sms/ipn` -- SSLCommerz IPN webhooks
  - `POST /api/payment/nagad/initiate`, `/confirm`, `/callback` -- Nagad payment
  - `GET /api/payment/verify/{orderId}` -- verify payment
  - `POST /api/payment/ssl/transaction` -- transaction query
  - `POST /api/payment/approval/status-change` -- CRM approval webhook
- **CRM integration:** Calls BTCL CRM API at `https://hcc.btcliptelephony.gov.bd/btcl` for approval and invoice creation
- **Package validation:** PBX (14/29/52 BDT), VBS (21/46/69 BDT), CC (9.5 BDT)
- **Profiles:** `local`, `btcl`
- **Source:** `/home/mustafa/telcobright-projects/RTC-Manager/PaymentGateWay/`

### csv_to_mysql (Data Import Utility)
- **Framework:** Spring Boot 3.3.1
- **Port:** 8808
- **Database:** MySQL (`telcobright`)
- **Dependencies:** OpenCSV, Apache HttpClient 5
- **Key endpoints:**
  - `POST /upload-csv` -- upload CSV file and import to MySQL
- **Source:** `/home/mustafa/telcobright-projects/RTC-Manager/csv_to_mysql/`

### Kafka (Campaign Task Dumper)
- **Framework:** Spring Boot 3.3.1
- **Port:** none (background service)
- **Database:** MySQL (`ofbiz`)
- **Dependencies:** Kafka clients 3.5.0, Quartz scheduler
- **Functionality:** Scheduled Kafka consumer that dumps campaign task data from Kafka to MySQL.
- **Source:** `/home/mustafa/telcobright-projects/RTC-Manager/Kafka/`

### WebSocket-SpringBoot
- **Framework:** Spring Boot 3.3.5
- **Port:** 8079
- **Dependencies:** Spring WebSocket, XMPP (Smack/Whack/Tinder), Jetty WebSocket client, WebFlux, ActiveJ, shared `util` library
- **Functionality:** WebSocket server with XMPP bridge for real-time communication (likely call notifications, presence).
- **Source:** `/home/mustafa/telcobright-projects/RTC-Manager/WebSocket-SpringBoot/`

### Shared Libraries

#### util
- **Artifact:** `com.tb:util:0.0.1-SNAPSHOT`
- **Used by:** FreeSwitchREST, FreeSwitchEsl, smsrest, WebSocket-SpringBoot
- **Contents:** Common entities, utilities, Kafka helpers. Build-only (no standalone port).
- **Source:** `/home/mustafa/telcobright-projects/RTC-Manager/util/`

#### partitioned-repo
- **Artifact:** `org.example:patitionedrepo:1.0-SNAPSHOT`
- **Used by:** event-aggregator, event-aggregator-incoming
- **Contents:** Hibernate entities for MySQL partitioned tables. Pure library, no Spring Boot.
- **Source:** `/home/mustafa/telcobright-projects/RTC-Manager/partitioned-repo/`

## External Connections

| Target                                      | Protocol        | Purpose                                              | Used By                          |
|---------------------------------------------|-----------------|------------------------------------------------------|----------------------------------|
| MySQL (various DBs per tenant)              | JDBC / R2DBC    | Primary datastore for all services                   | All services                     |
| PostgreSQL (`fusionpbx` at 103.95.96.100)   | JDBC            | FusionPBX domain/extension/gateway management        | ConfigManager, Security, FreeSwitchREST |
| MySQL (`ofbiz`)                             | JDBC            | Legacy OFBiz data, SMS tasks                         | smsprocessor, Kafka, sms-delivery-report |
| MySQL (`dolibarr`)                          | JDBC            | Dolibarr ERP for billing/invoicing                   | erp                              |
| MySQL (`tenant_master`)                     | R2DBC           | Gateway auth/routing metadata                        | API-Gateway                      |
| Apache Kafka (localhost:9092 / 29092)       | Kafka protocol  | Event streaming: SMS events, CDR, config CDC, payment | ConfigManager, FreeSwitchEsl, smsprocessor, event-aggregator*, erp, Grpc-Router, PaymentGateWay |
| FreeSWITCH (ESL)                            | TCP/ESL         | Call control, event subscription, conference mgmt    | FreeSwitchEsl, FreeSwitchREST    |
| Grpc-Router (45.251.228.3:9000)             | gRPC            | SMS routing and delivery                             | smsrest, Apache-Flink            |
| Redis                                       | Lettuce         | State management for stream processing               | Apache-Flink                     |
| SSLCommerz API                              | HTTPS           | Payment processing                                   | PaymentGateWay                   |
| Nagad API                                   | HTTPS           | Mobile payment processing                            | PaymentGateWay                   |
| BTCL CRM (`hcc.btcliptelephony.gov.bd`)    | HTTPS           | Approval and invoice creation                        | PaymentGateWay                   |
| Eureka (localhost:8761)                     | HTTP            | Service registration and discovery                   | All Spring Boot services         |

## Data Flow

### Voice Call Flow
```
Incoming SIP -> FreeSWITCH -> ESL events -> FreeSwitchEsl (port 5070)
                                              |
                                              +--> Kafka (call events)
                                              |      |
                                              |      +--> event-aggregator -> MySQL CDR / CSV
                                              |      +--> event-aggregator-incoming -> MySQL CDR / CSV
                                              |
                                              +--> gRPC -> Grpc-Router (call state machine)
                                              |
                                              +--> WebSocket -> WebSocket-SpringBoot -> UI
```

### SMS Flow
```
REST API (smsrest port 6080) -> gRPC -> Grpc-Router (port 9000)
                                           |
                                           +--> Kafka topic -> smsprocessor (port 6071) -> external SMSC
                                           |
                                           +--> Kafka topic -> event-aggregator -> CDR (MySQL/CSV)
                                           |
                                           +--> Kafka topic -> sms-delivery-report -> delivery status updates
```

### Payment Flow
```
UI -> API-Gateway (8001) -> PaymentGateWay (8081)
                               |
                               +--> SSLCommerz/Nagad API (initiate)
                               |
                               <-- IPN webhook (validate + confirm)
                               |
                               +--> Kafka (payment_approved) -> downstream services
                               |
                               +--> BTCL CRM API (approval + invoice)
```

### Config Change Propagation
```
ConfigManager (7071) -> Kafka (telcobright_all_tables CDC topic) -> Grpc-Router (config reload)
```

## Configuration

### Profiles Per Tenant

Services support multiple tenant profiles via `spring.profiles.active`:

| Profile   | Tenant/Environment                 |
|-----------|------------------------------------|
| `local`   | Local development                  |
| `ccl76`   | CCL client (76 series)             |
| `ccl` / `ccl98` | CCL alternate deployments    |
| `btcl`    | BTCL (Bangladesh Telephone Company)|
| `link3`   | Link3 / L3Venture                  |
| `dev`     | Development/testing                |
| `kafka-fix` | ConfigManager Kafka fix profile |

Each profile provides environment-specific values for:
- Database connection URLs and credentials
- Kafka bootstrap servers
- Service URLs (Eureka, Security, ConfigManager, ESL)
- R2DBC connection strings (API-Gateway)

### Inter-Service Authentication

- **JWT tokens:** Gateway validates via Security service
- **System access key:** `system-access-key=kshfsihaifkskfhiosks3u128#$%&` (shared between ConfigManager, FreeSwitchEsl, smsrest)
- **Internal secret header:** `security.secret-header-value=oi-secur1ty-u-better=0bey-ur-Mast3r` (FreeSwitchREST, Security, smsrest)

### Port Summary

| Service                    | Port  |
|----------------------------|-------|
| Eureka-Server              | 8761  |
| API-Gateway                | 8001  |
| Security                   | 8080  |
| ConfigManager              | 7071  |
| FreeSwitchREST             | 5071  |
| FreeSwitchEsl              | 5070  |
| smsrest                    | 6080  |
| smsprocessor               | 6071  |
| erp                        | 8076  |
| PaymentGateWay             | 8081  |
| csv_to_mysql               | 8808  |
| WebSocket-SpringBoot       | 8079  |
| Grpc-Router (HTTP)         | 9001  |
| Grpc-Router (gRPC)         | 9000  |

## Deployment

### Deploy Scripts

Located at `/home/mustafa/telcobright-projects/RTC-Manager/Tools/`:

| Script                   | Target                                |
|--------------------------|---------------------------------------|
| `ccl_76_deploy.sh`       | CCL 76 deployment                     |
| `ccl_98_deploy.sh`       | CCL 98 deployment                     |
| `btcl_pbx_deploy.sh`     | BTCL PBX (114.130.145.75:40001)       |
| `btcl_vbs_deploy.sh`     | BTCL Voice Broadcasting               |
| `btcl_cc_deploy.sh`      | BTCL Contact Center                   |
| `btcl_services_deploy.sh`| BTCL services                         |

### Standard Deployment Order

From `deployment_instruction.txt` and deploy scripts:

1. `Eureka-Server` (port 8761)
2. `API-Gateway` (port 8001)
3. `ConfigManager` (port 7071)
4. `Security` (port 8080)
5. `util` (build only, dependency for other services)
6. `FreeSwitchREST` (port 5071)
7. `smsrest` (port 6080)
8. `PaymentGateWay` (port 8081, optional)

### Build and Run

Each service is built with Maven and runs as a standalone JAR:
```bash
cd $BASE/<ServiceName>
mvn clean install -DskipTests
nohup java -Xmx384m -jar target/*.jar > $BASE/<ServiceName>.log 2>&1 &
```

Memory allocation: Eureka/Gateway at 384m, ConfigManager at 512m, others at default.

### Target Servers

| Server            | Port  | Container       | Path                          |
|-------------------|-------|-----------------|-------------------------------|
| 114.130.145.75    | 40001 | SoftSwitch (LXC)| /home/sbc_voice/RTC-Manager   |
| 114.130.145.75    | 40002 | ContactCenter   | /home/sbc_voice/RTC-Manager   |

## Service Discovery

All Spring Boot services register with **Netflix Eureka** at `http://localhost:8761/eureka`. The API-Gateway uses Eureka's service discovery locator (`spring.cloud.gateway.discovery.locator.enabled=true`) to automatically route requests based on registered service names. Service names in Eureka are uppercase (e.g., `AUTHENTICATION`, `FREESWITCHREST`, `SMSREST`, `CONFIGMANAGER`).

The Quarkus services (event-aggregator, event-aggregator-incoming, sms-delivery-report, Grpc-Router) do **not** register with Eureka. They communicate via direct Kafka topics or hardcoded gRPC endpoints.

## Integration Points with Other Projects

### routesphere / routesphere-core
- RTC-Manager's smsrest and smsprocessor provide the REST API and Kafka infrastructure for SMS operations
- The Grpc-Router gRPC service at `45.251.228.3:9000` is the same protocol/schema used by routesphere's sigtran SMS pipeline
- The `sms.proto` protobuf definition is shared across smsrest, Grpc-Router, and Apache-Flink
- ConfigManager CDC topics (`telcobright_all_tables`) feed the same Kafka cluster used by routesphere's config-manager

### config-manager (routesphere config-manager)
- Both projects have a ConfigManager service at port 7071
- RTC-Manager's ConfigManager is the Spring Boot version serving RTC-Manager services
- Both use Kafka CDC for configuration change propagation

### FusionPBX
- FreeSwitchREST and Security directly manage FusionPBX PostgreSQL database (`fusionpbx` at `103.95.96.100:5432`)
- FreeSwitchREST provides full PBX lifecycle management: extensions, gateways, domains, dialplans, recordings

### Dolibarr ERP
- The erp service connects to Dolibarr MySQL database for billing/invoicing integration

### BTCL CRM
- PaymentGateWay integrates with BTCL's hosted CRM at `hcc.btcliptelephony.gov.bd` for service approval workflows and invoice generation
