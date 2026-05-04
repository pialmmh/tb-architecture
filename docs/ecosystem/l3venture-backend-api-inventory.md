# L3venture Backend API Inventory

Comprehensive REST endpoint catalog for the Softswitch Dashboard backend services.
All services register with Eureka and are accessed through the API-Gateway (port 8001).

---

## API-Gateway (port 8001, service name: `gateway`)

The gateway uses **Spring Cloud Gateway with Eureka discovery locator** (`spring.cloud.gateway.discovery.locator.enabled=true`).
Routes are auto-generated from Eureka service names as path prefixes:

| Frontend Path Prefix | Backend Service | Eureka Name | Port |
|---|---|---|---|
| `/FREESWITCHREST/` | FreeSwitchREST | `FreeswitchRest` | 5071 |
| `/SMSREST/` | smsrest | `smsrest` | 6080 |
| `/AUTHENTICATION/` | Security | `AUTHENTICATION` | 8080 |

**Gateway also provides:**
- Token validation: calls `http://localhost:8080/auth/validateAccess?token=` on the Security service
- R2DBC connection to `tenant_master` MySQL database (link3 profile: `l3ventures_sms`)
- Per-policy authorization (Admin, User, Reseller, BTRC, WebRTC, SMS Sender, ReadOnly, DenyAll)
- Audit logging (excludes high-frequency dashboard polling endpoints)

---

## FREESWITCHREST (port 5071, service name: `FreeswitchRest`)

Spring Boot 3.2.5, Java 21. Dual datasource: MySQL (`l3ventures_sms`) + PostgreSQL (FusionPBX).
Dependencies: Spring Data JPA, Spring Kafka, Chronicle Queue, FreeSWITCH ESL client, Apache POI, OpenPDF, Spring Mail, WebSocket, SpringDoc OpenAPI.

### Link3 Config Highlights
- DB: MySQL at `10.10.199.10:3306/l3ventures_sms`
- ESL: `127.0.0.1:8021` (FreeSWITCH Event Socket)
- PostgreSQL: FusionPBX at `114.130.145.82:5432/fusionpbx`
- Kafka: `localhost:9092`, consumer group `tenant_creator`

---

### Controller: AccountController
**File:** `freeswitch/controller/AccountController.java`
**Base path:** _(none)_

| Method | Path | Description |
|---|---|---|
| POST | `/create-account` | Create account |
| POST | `/recharge` | Recharge account |

### Controller: ActiveCallController
**File:** `freeswitch/controller/ActiveCallController.java`
**Base path:** `/api/v1/active-calls`

| Method | Path | Description |
|---|---|---|
| POST | `/api/v1/active-calls/list` | List active calls |
| POST | `/api/v1/active-calls/details` | Get active call details |
| POST | `/api/v1/active-calls/hangup` | Hang up a call |
| POST | `/api/v1/active-calls/transfer` | Transfer a call |
| POST | `/api/v1/active-calls/hold` | Hold a call |
| POST | `/api/v1/active-calls/mute` | Mute a call |
| POST | `/api/v1/active-calls/conference` | Bridge call into conference |

### Controller: AuditLogController
**File:** `freeswitch/controller/audit/AuditLogController.java`
**Base path:** _(none)_

| Method | Path | Description |
|---|---|---|
| POST | `/get-audit-logs` | Retrieve audit logs |

### Controller: AuthIpAuthenticationController
**File:** `freeswitch/controller/AuthIpAuthenticationController.java`
**Base path:** `/api/ip-auth`

| Method | Path | Description |
|---|---|---|
| POST | `/api/ip-auth/create` | Create IP auth entry |
| POST | `/api/ip-auth/get-all` | List all IP auth entries |
| POST | `/api/ip-auth/get-by-id` | Get IP auth by ID |
| POST | `/api/ip-auth/update` | Update IP auth entry |
| POST | `/api/ip-auth/delete` | Delete IP auth entry |
| POST | `/api/ip-auth/get-ip-by-idpartner` | Get IPs by partner ID |
| POST | `/api/ip-auth/get-ip-by-idauthuser` | Get IPs by auth user ID |

### Controller: BalanceController
**File:** `freeswitch/controller/BalanceController.java`
**Base path:** _(none)_

| Method | Path | Description |
|---|---|---|
| POST | `/topup` | Top-up balance |
| POST | `/check-balance` | Check balance |

### Controller: BroadcastController
**File:** `freeswitch/controller/BroadcastController.java`
**Base path:** `/api/broadcast`

| Method | Path | Description |
|---|---|---|
| POST | `/api/broadcast/play-audio` | Upload & play audio broadcast (multipart) |

### Controller: CallbackController
**File:** `freeswitch/controller/CallbackController.java`
**Base path:** `/api/v1/callback`

| Method | Path | Description |
|---|---|---|
| POST | `/api/v1/callback/config/create` | Create callback config |
| POST | `/api/v1/callback/config/list` | List callback configs |
| POST | `/api/v1/callback/config/get` | Get callback config |
| POST | `/api/v1/callback/config/update` | Update callback config |
| POST | `/api/v1/callback/config/delete` | Delete callback config |
| POST | `/api/v1/callback/config/toggle` | Toggle callback config |
| POST | `/api/v1/callback/queue/create` | Create callback queue entry |
| POST | `/api/v1/callback/queue/list` | List callback queue |
| POST | `/api/v1/callback/queue/get` | Get callback queue entry |
| POST | `/api/v1/callback/queue/cancel` | Cancel callback queue entry |
| POST | `/api/v1/callback/queue/retry` | Retry callback queue entry |
| POST | `/api/v1/callback/install` | Install callback dialplan |

### Controller: CallBlockController
**File:** `freeswitch/controller/CallBlockController.java`
**Base path:** `/api/v1/call-block`

| Method | Path | Description |
|---|---|---|
| POST | `/api/v1/call-block/list-by-domain` | List call blocks by domain |
| POST | `/api/v1/call-block/create` | Create call block rule |
| POST | `/api/v1/call-block/update` | Update call block rule |
| POST | `/api/v1/call-block/delete` | Delete call block rule |
| POST | `/api/v1/call-block/get-by-uuid` | Get call block by UUID |
| POST | `/api/v1/call-block/toggle` | Toggle call block |

### Controller: CallBroadcastController
**File:** `freeswitch/controller/CallBroadcastController.java`
**Base path:** `/api/call-broadcast`

| Method | Path | Description |
|---|---|---|
| POST | `/api/call-broadcast/v1/list` | List call broadcasts |
| POST | `/api/call-broadcast/v1/details` | Get broadcast details |
| POST | `/api/call-broadcast/v1/create` | Create call broadcast |
| POST | `/api/call-broadcast/v1/update` | Update call broadcast |
| POST | `/api/call-broadcast/v1/delete` | Delete call broadcast |
| POST | `/api/call-broadcast/v1/start` | Start call broadcast |
| POST | `/api/call-broadcast/v1/stop` | Stop call broadcast |
| POST | `/api/call-broadcast/v1/upload-leads` | Upload broadcast leads |

### Controller: CallCenterController
**File:** `freeswitch/controller/CallCenterController.java`
**Base path:** `/api/call-center`

| Method | Path | Description |
|---|---|---|
| POST | `/api/call-center/v1/queues/list` | List call center queues |
| POST | `/api/call-center/v1/queues/details` | Get queue details |
| POST | `/api/call-center/v1/queues/status` | Get queue status |
| POST | `/api/call-center/v1/queues/live` | Get live queue data |
| POST | `/api/call-center/v1/eavesdrop` | Eavesdrop on call |
| POST | `/api/call-center/v1/queues/create` | Create queue |
| POST | `/api/call-center/v1/queues/update` | Update queue |
| POST | `/api/call-center/v1/queues/delete` | Delete queue |
| POST | `/api/call-center/v1/agents/list` | List agents |
| POST | `/api/call-center/v1/agents/details` | Get agent details |
| POST | `/api/call-center/v1/agents/status` | Get agent status |
| POST | `/api/call-center/v1/agents/set-status` | Set agent status |
| POST | `/api/call-center/v1/agents/set-state` | Set agent state |
| POST | `/api/call-center/v1/agents/create` | Create agent |
| POST | `/api/call-center/v1/agents/update` | Update agent |
| POST | `/api/call-center/v1/agents/delete` | Delete agent |
| POST | `/api/call-center/v1/agents/stats` | Get agent statistics |
| POST | `/api/call-center/v1/tiers/list` | List tiers |
| POST | `/api/call-center/v1/tiers/add` | Add tier |
| POST | `/api/call-center/v1/tiers/remove` | Remove tier |

### Controller: CallForwardController
**File:** `freeswitch/controller/CallForwardController.java`
**Base path:** `/api/v1/call-forward`

| Method | Path | Description |
|---|---|---|
| POST | `/api/v1/call-forward/list-by-domain` | List call forwards by domain |
| POST | `/api/v1/call-forward/get-by-extension` | Get forwards by extension |
| POST | `/api/v1/call-forward/update` | Update call forward |
| POST | `/api/v1/call-forward/toggle-dnd` | Toggle Do Not Disturb |
| POST | `/api/v1/call-forward/toggle-forward-all` | Toggle forward all |

### Controller: CallPermissionsController
**File:** `freeswitch/controller/CallPermissionsController.java`
**Base path:** `/api/v1/call-permissions`

| Method | Path | Description |
|---|---|---|
| POST | `/api/v1/call-permissions/list-by-domain` | List permissions by domain |
| POST | `/api/v1/call-permissions/update` | Update call permissions |

### Controller: CallRecordingController
**File:** `freeswitch/controller/CallRecordingController.java`
**Base path:** `/api/recordings`

| Method | Path | Description |
|---|---|---|
| POST | `/api/recordings/get-recordings` | Get recordings (legacy) |
| POST | `/api/recordings/search-recordings` | Search recordings (legacy) |
| POST | `/api/recordings/download-zip` | Download recordings as ZIP |
| POST | `/api/recordings/auto-load` | Auto-load recordings |
| POST | `/api/recordings/stream` | Stream a recording |
| POST | `/api/recordings/v1/list` | List recordings (v1) |
| POST | `/api/recordings/v1/list-by-domain` | List by domain (v1) |
| POST | `/api/recordings/v1/get-by-uuid` | Get by UUID (v1) |
| POST | `/api/recordings/v1/delete` | Delete recording (v1) |
| POST | `/api/recordings/v1/get-by-call-uuid` | Get by call UUID (v1) |
| POST | `/api/recordings/v1/get-by-sip-call-id` | Get by SIP call ID (v1) |
| POST | `/api/recordings/v1/bulk-info` | Bulk recording info (v1) |
| POST | `/api/recordings/v1/bulk-download` | Bulk download (v1) |
| POST | `/api/recordings/v1/download` | Download single (v1) |

### Controller: CallSrcController
**File:** `freeswitch/controller/CallSrcController.java`
**Base path:** _(none)_

| Method | Path | Description |
|---|---|---|
| POST | `/create-callsrc` | Create call source |
| POST | `/get-callsrcs` | Get call sources (paginated) |
| POST | `/update-callsrc` | Update call source |

### Controller: CdrApiController
**File:** `freeswitch/controller/CdrApiController.java`
**Base path:** `/api/v1/cdr`

| Method | Path | Description |
|---|---|---|
| POST | `/api/v1/cdr/list-by-domain` | List CDRs by domain (PBX v1) |
| POST | `/api/v1/cdr/stats` | CDR statistics (PBX v1) |
| POST | `/api/v1/cdr/get-by-uuid` | Get CDR by UUID (PBX v1) |
| GET | `/api/v1/cdr/hangup-causes` | List hangup cause codes |

### Controller: CdrController
**File:** `freeswitch/controller/CdrController.java`
**Base path:** `/api/cdr`

| Method | Path | Description |
|---|---|---|
| POST | `/api/cdr/cdrByFilterForAdmin` | CDR by filter (admin) |
| POST | `/api/cdr/cdrByFilterExportForAdmin` | Export CDR (admin, octet-stream) |
| POST | `/api/cdr/cdrByFilterForUser` | CDR by filter (user) |
| POST | `/api/cdr/cdrByFilterExportForUser` | Export CDR (user, octet-stream) |

### Controller: ConcurentCallAnalyticsController
**File:** `freeswitch/controller/ConcurentCallAnalyticsController.java`
**Base path:** `/analytics`

| Method | Path | Description |
|---|---|---|
| POST | `/analytics/graph-data` | Concurrent call graph data |
| POST | `/analytics/graph-data/csv` | Export concurrent call data as CSV |

### Controller: ConferenceController
**File:** `freeswitch/controller/ConferenceController.java`
**Base path:** `/api/v1/conferences`

| Method | Path | Description |
|---|---|---|
| POST | `/api/v1/conferences/create` | Create conference room |
| POST | `/api/v1/conferences/list-by-domain` | List conferences by domain |
| POST | `/api/v1/conferences/get-by-uuid` | Get conference by UUID |
| POST | `/api/v1/conferences/update` | Update conference |
| POST | `/api/v1/conferences/delete` | Delete conference |
| POST | `/api/v1/conferences/toggle` | Toggle conference |

### Controller: ContactController
**File:** `freeswitch/controller/ContactController.java`
**Base path:** `/contact`

| Method | Path | Description |
|---|---|---|
| POST | `/contact/create-contact` | Create contact |
| POST | `/contact/get-contact` | Get single contact |
| POST | `/contact/get-contacts` | Get all contacts |
| POST | `/contact/delete-contact` | Delete contact |
| POST | `/contact/update-contact` | Update contact |

### Controller: DashboardControllerAdmin
**File:** `freeswitch/controller/DashboardControllerAdmin.java`
**Base path:** `/admin/DashBoard`

| Method | Path | Description |
|---|---|---|
| POST | `/admin/DashBoard/getTotalCall` | Total calls (admin) |
| POST | `/admin/DashBoard/getOutgoingCall` | Outgoing calls (admin) |
| POST | `/admin/DashBoard/getIncomingCall` | Incoming calls (admin) |
| POST | `/admin/DashBoard/getMissedCall` | Missed calls (admin) |
| POST | `/admin/DashBoard/getIntervalWiseCall` | Interval-wise call data (admin) |
| POST | `/admin/DashBoard/get-partner-balances` | Partner balances (paginated) |
| POST | `/admin/DashBoard/get-partner-balances-unified` | Unified partner balances (prepaid/postpaid) |
| POST | `/admin/DashBoard/system-info` | System info (CPU, memory, etc.) |
| POST | `/admin/DashBoard/top5PartnerCallCounts` | Top 5 partner call counts |
| POST | `/admin/DashBoard/partner/validate` | Validate partner |

### Controller: DashboardControllerUser
**File:** `freeswitch/controller/DashboardControllerUser.java`
**Base path:** `/user/DashBoard`

| Method | Path | Description |
|---|---|---|
| POST | `/user/DashBoard/getTotalCall` | Total calls (user) |
| POST | `/user/DashBoard/getOutgoingCall` | Outgoing calls (user) |
| POST | `/user/DashBoard/getIncomingCall` | Incoming calls (user) |
| POST | `/user/DashBoard/getMissedCall` | Missed calls (user) |
| POST | `/user/DashBoard/getIntervalWiseCall` | Interval-wise call data (user) |
| POST | `/user/DashBoard/getTopupBalanceForUser` | Topup balance (user) |
| POST | `/user/DashBoard/getBalance` | Get balance (user) |
| POST | `/user/DashBoard/processPostpaidPayment` | Process postpaid payment |
| POST | `/user/DashBoard/addPostpaidUsage` | Add postpaid usage |

### Controller: DatabaseHealthController
**File:** `freeswitch/controller/DatabaseHealthController.java`
**Base path:** `/api/v1/health`

| Method | Path | Description |
|---|---|---|
| POST | `/api/v1/health/postgres` | Check PostgreSQL health |

### Controller: DestinationController
**File:** `freeswitch/controller/DestinationController.java`
**Base path:** `/api/v1/destinations`

| Method | Path | Description |
|---|---|---|
| POST | `/api/v1/destinations/create` | Create destination |
| POST | `/api/v1/destinations/list` | List all destinations |
| POST | `/api/v1/destinations/list-by-domain` | List by domain |
| POST | `/api/v1/destinations/list-by-type` | List by type |
| POST | `/api/v1/destinations/list-by-domain-type` | List by domain and type |
| POST | `/api/v1/destinations/get-by-uuid` | Get by UUID |
| POST | `/api/v1/destinations/get-by-number` | Get by number |
| POST | `/api/v1/destinations/update` | Update destination |
| POST | `/api/v1/destinations/delete` | Delete destination |
| POST | `/api/v1/destinations/toggle` | Toggle destination |

### Controller: DeviceTokenController
**File:** `freeswitch/controller/DeviceTokenController.java`
**Base path:** `/device-token`

| Method | Path | Description |
|---|---|---|
| POST | `/device-token/save` | Save device token (push notifications) |
| POST | `/device-token/get-device_token` | Get device token |

### Controller: DialplanController
**File:** `freeswitch/controller/DialplanController.java`
**Base path:** _(none)_

| Method | Path | Description |
|---|---|---|
| POST | `/get-dialplan` | Get dialplan by callsrc ID |
| POST | `/create-dialplan` | Create dialplan |
| POST | `/get-dialplan-by-id` | Get dialplan by ID |
| POST | `/get-dialplans` | Get all dialplans (paginated) |
| POST | `/update-dialplan` | Update dialplan |
| POST | `/delete-dialplan` | Delete dialplan |
| POST | `/add-route-to-dialplan` | Add route to dialplan |
| POST | `/delete-dialplan-route` | Delete dialplan route |

### Controller: DialPlanPrefixController
**File:** `freeswitch/controller/DialPlanPrefixController.java`
**Base path:** _(none)_

| Method | Path | Description |
|---|---|---|
| POST | `/create-dialplan-prefix` | Create dialplan prefix |
| POST | `/update-dialplan-prefix` | Update dialplan prefix |
| POST | `/get-dialplan-prefix` | Get single dialplan prefix |
| POST | `/delete-dialplan-prefix` | Delete dialplan prefix |
| POST | `/dialplan-prefix/exists` | Check if prefix exists |
| POST | `/get-dialplan-prefixes` | Get all prefixes (paginated) |
| POST | `/get-dialplan-prefixes-grouped` | Get prefixes grouped |

### Controller: Did__Controller
**File:** `freeswitch/controller/Did__Controller.java`
**Base path:** _(none)_
**Manages:** DID pools, DID numbers, DID assignments

| Method | Path | Description |
|---|---|---|
| POST | `/cause-error` | Test: cause error |
| POST | `/cause-unexpected-error` | Test: cause unexpected error |
| POST | `/get-did-pool` | Get single DID pool |
| POST | `/get-did-pools` | Get all DID pools |
| POST | `/update-did-pool` | Update DID pool |
| POST | `/create-did-pool` | Create DID pool |
| POST | `/delete-did-pool` | Delete DID pool |
| POST | `/get-did-number` | Get single DID number |
| POST | `/get-did-numbers` | Get all DID numbers |
| POST | `/get-unassigned-did-numbers` | Get unassigned DID numbers |
| POST | `/update-did-number` | Update DID number |
| POST | `/delete-did-number` | Delete DID number |
| POST | `/create-did-number` | Create DID number |
| POST | `/create-did-number-excel` | Create DID numbers from Excel |
| POST | `/get-did-numbers-by-pool` | Get DID numbers by pool |
| POST | `/create-did-assignment` | Create DID assignment |
| POST | `/update-did-assignment` | Update DID assignment |
| POST | `/delete-did-assignment` | Delete DID assignment |
| POST | `/get-did-assignment` | Get single DID assignment |
| POST | `/get-did-assignment-by-partner-id` | Get assignments by partner ID |
| POST | `/get-did-assignment-history-by-partner-id` | Get assignment history by partner |
| POST | `/get-did-assignments` | Get all DID assignments |
| POST | `/create-did-assign-from-csv` | Create DID assignments from CSV |
| POST | `/get-did-assignment-history` | Get DID assignment history |

### Controller: DigitFilterPlanController
**File:** `freeswitch/controller/DigitFilterPlanController.java`
**Base path:** `/api/plans`

| Method | Path | Description |
|---|---|---|
| POST | `/api/plans/list` | List digit filter plans |
| POST | `/api/plans/create` | Create digit filter plan |
| POST | `/api/plans/update` | Update digit filter plan |
| POST | `/api/plans/delete` | Delete digit filter plan |

### Controller: DigitFilterRuleController
**File:** `freeswitch/controller/DigitFilterRuleController.java`
**Base path:** `/api/digit-filter-rules`

| Method | Path | Description |
|---|---|---|
| POST | `/api/digit-filter-rules/create` | Create rule |
| POST | `/api/digit-filter-rules/list` | List rules |
| POST | `/api/digit-filter-rules/get` | Get single rule |
| POST | `/api/digit-filter-rules/update` | Update rule |
| POST | `/api/digit-filter-rules/delete` | Delete rule |
| POST | `/api/digit-filter-rules/toggle-status` | Toggle rule status |
| POST | `/api/digit-filter-rules/actions/create` | Create rule action |
| POST | `/api/digit-filter-rules/actions/list` | List rule actions |
| POST | `/api/digit-filter-rules/actions/delete` | Delete rule action |
| POST | `/api/digit-filter-rules/actions/update-priority` | Update action priority |
| POST | `/api/digit-filter-rules/actions/toggle-status` | Toggle action status |

### Controller: DomainController
**File:** `freeswitch/controller/DomainController.java`
**Base path:** `/api/v1/domains`

| Method | Path | Description |
|---|---|---|
| POST | `/api/v1/domains/create` | Create domain |
| POST | `/api/v1/domains/list` | List all domains |
| POST | `/api/v1/domains/get-by-uuid` | Get domain by UUID |
| POST | `/api/v1/domains/get-by-name` | Get domain by name |
| POST | `/api/v1/domains/update` | Update domain |
| POST | `/api/v1/domains/delete` | Delete domain |
| POST | `/api/v1/domains/toggle` | Toggle domain |

### Controller: EmailController
**File:** `freeswitch/controller/EmailController.java`
**Base path:** `/api/v1/email`

| Method | Path | Description |
|---|---|---|
| POST | `/api/v1/email/send` | Send email |

### Controller: ExtensionController
**File:** `freeswitch/controller/ExtensionController.java`
**Base path:** `/api/v1/extensions`

| Method | Path | Description |
|---|---|---|
| POST | `/api/v1/extensions/create` | Create extension |
| POST | `/api/v1/extensions/list` | List all extensions |
| POST | `/api/v1/extensions/list-by-domain` | List by domain |
| POST | `/api/v1/extensions/get-by-uuid` | Get by UUID |
| POST | `/api/v1/extensions/get-by-number` | Get by number |
| POST | `/api/v1/extensions/update` | Update extension |
| POST | `/api/v1/extensions/delete` | Delete extension |
| POST | `/api/v1/extensions/toggle` | Toggle extension |

### Controller: Fail2BanController
**File:** `freeswitch/controller/Fail2BanController.java`
**Base path:** `/api/fail2ban`

| Method | Path | Description |
|---|---|---|
| POST | `/api/fail2ban/banned-ips` | List banned IPs |
| POST | `/api/fail2ban/ban-ip` | Ban an IP |
| POST | `/api/fail2ban/unban-ip` | Unban an IP |
| POST | `/api/fail2ban/config/get` | Get Fail2Ban config |
| POST | `/api/fail2ban/config/update` | Update Fail2Ban config |

### Controller: FsController
**File:** `freeswitch/controller/FsController.java`
**Base path:** _(none)_
**Manages:** FreeSWITCH live data via ESL

| Method | Path | Description |
|---|---|---|
| POST | `/profile-path` | Get SIP profile path |
| POST | `/getConcurrentCall` | Get concurrent call count |
| POST | `/get-partner-details` | Get partner details |
| POST | `/get-btcl-calls` | Get BTCL calls |
| POST | `/get-outgoing-calls` | Get outgoing calls |
| POST | `/get-partner-concurrent-call-limit` | Get partner concurrent call limit |
| POST | `/concurrent-call-profile` | Concurrent calls by SIP profile |
| POST | `/concurrent-call-profile-partner` | Concurrent calls by profile + partner |
| POST | `/concurrent-call-partner-wise` | Concurrent calls partner-wise |
| POST | `/admin-live-calls-summary` | Admin live calls summary |
| POST | `/user-live-calls-summary` | User live calls summary |

### Controller: FusionPbxContactController
**File:** `freeswitch/controller/FusionPbxContactController.java`
**Base path:** `/api/v1/fusionpbx-contact`

| Method | Path | Description |
|---|---|---|
| POST | `/api/v1/fusionpbx-contact/list-by-domain` | List contacts by domain |
| POST | `/api/v1/fusionpbx-contact/get-by-uuid` | Get contact by UUID |
| POST | `/api/v1/fusionpbx-contact/create` | Create contact |
| POST | `/api/v1/fusionpbx-contact/update` | Update contact |
| POST | `/api/v1/fusionpbx-contact/delete` | Delete contact |
| POST | `/api/v1/fusionpbx-contact/list-by-extension` | List contacts by extension |

### Controller: FusionPbxIvrController
**File:** `freeswitch/controller/FusionPbxIvrController.java`
**Base path:** `/api/v1/fusionpbx-ivr`

| Method | Path | Description |
|---|---|---|
| POST | `/api/v1/fusionpbx-ivr/list-by-domain` | List IVRs by domain |
| POST | `/api/v1/fusionpbx-ivr/get-by-uuid` | Get IVR by UUID |
| POST | `/api/v1/fusionpbx-ivr/create` | Create IVR |
| POST | `/api/v1/fusionpbx-ivr/update` | Update IVR |
| POST | `/api/v1/fusionpbx-ivr/delete` | Delete IVR |

### Controller: GatewayController
**File:** `freeswitch/controller/GatewayController.java`
**Base path:** `/api/v1/gateways`

| Method | Path | Description |
|---|---|---|
| POST | `/api/v1/gateways/list` | List all gateways |
| POST | `/api/v1/gateways/list-by-domain` | List by domain |
| POST | `/api/v1/gateways/details` | Get gateway details |
| POST | `/api/v1/gateways/get-by-uuid` | Get by UUID |
| POST | `/api/v1/gateways/create` | Create gateway |
| POST | `/api/v1/gateways/update` | Update gateway |
| POST | `/api/v1/gateways/delete` | Delete gateway |
| POST | `/api/v1/gateways/toggle` | Toggle gateway |
| GET | `/api/v1/gateways/profiles` | List SIP profiles |
| GET | `/api/v1/gateways/transports` | List transports |

### Controller: InboundRouteController
**File:** `freeswitch/controller/InboundRouteController.java`
**Base path:** `/api/v1/inbound-routes`

| Method | Path | Description |
|---|---|---|
| POST | `/api/v1/inbound-routes/create` | Create inbound route |
| POST | `/api/v1/inbound-routes/list-by-domain` | List by domain |
| POST | `/api/v1/inbound-routes/get-by-uuid` | Get by UUID |
| POST | `/api/v1/inbound-routes/update` | Update inbound route |
| POST | `/api/v1/inbound-routes/delete` | Delete inbound route |
| POST | `/api/v1/inbound-routes/toggle` | Toggle inbound route |

### Controller: IvrMenuController
**File:** `freeswitch/controller/IvrMenuController.java`
**Base path:** `/api/v1/ivr-menus`

| Method | Path | Description |
|---|---|---|
| POST | `/api/v1/ivr-menus/create` | Create IVR menu |
| POST | `/api/v1/ivr-menus/list` | List all IVR menus |
| POST | `/api/v1/ivr-menus/list-by-domain` | List by domain |
| POST | `/api/v1/ivr-menus/get-by-uuid` | Get by UUID |
| POST | `/api/v1/ivr-menus/get-by-extension` | Get by extension |
| POST | `/api/v1/ivr-menus/update` | Update IVR menu |
| POST | `/api/v1/ivr-menus/delete` | Delete IVR menu |
| POST | `/api/v1/ivr-menus/toggle` | Toggle IVR menu |
| POST | `/api/v1/ivr-menus/add-option` | Add menu option |
| POST | `/api/v1/ivr-menus/delete-option` | Delete menu option |

### Controller: MaximumConcurrentCallController
**File:** `freeswitch/controller/MaximumConcurrentCallController.java`
**Base path:** `/mcc-reports`

| Method | Path | Description |
|---|---|---|
| POST | `/mcc-reports/max-call-periodically` | Max concurrent call report |
| POST | `/mcc-reports/export/csv` | Export as CSV |
| POST | `/mcc-reports/export/excel` | Export as Excel |

### Controller: MnpController
**File:** `freeswitch/controller/MnpController.java`
**Base path:** _(none)_

| Method | Path | Description |
|---|---|---|
| POST | `/number` | MNP number lookup |

### Controller: MnpPullController
**File:** `freeswitch/controller/MnpPullController.java`
**Base path:** `/api/mnp_pull`

| Method | Path | Description |
|---|---|---|
| POST | `/api/mnp_pull` | Pull MNP data |
| POST | `/api/mnp_pull/file-count` | MNP file count |
| POST | `/api/mnp_pull/export-file-list` | Export MNP file list |

### Controller: OtpController
**File:** `freeswitch/controller/otp/OtpController.java`
**Base path:** `/otp`

| Method | Path | Description |
|---|---|---|
| POST | `/otp/send` | Send OTP via SMS |
| POST | `/otp/varify` | Verify OTP |
| POST | `/otp/email/send` | Send OTP via email |
| POST | `/otp/email/verify` | Verify email OTP |

### Controller: OutboundRouteController
**File:** `freeswitch/controller/OutboundRouteController.java`
**Base path:** `/api/v1/outbound-routes`

| Method | Path | Description |
|---|---|---|
| POST | `/api/v1/outbound-routes/list` | List outbound routes |
| POST | `/api/v1/outbound-routes/details` | Get route details |
| POST | `/api/v1/outbound-routes/create` | Create outbound route |
| POST | `/api/v1/outbound-routes/update` | Update outbound route |
| POST | `/api/v1/outbound-routes/delete` | Delete outbound route |
| POST | `/api/v1/outbound-routes/toggle` | Toggle outbound route |

### Controller: PackageController
**File:** `freeswitch/controller/PackageController.java`
**Base path:** `/package`
**Manages:** Packages, package items, purchases, topups, postpaid credit

| Method | Path | Description |
|---|---|---|
| POST | `/package/create-package` | Create package |
| POST | `/package/active-inactive` | Toggle package active/inactive |
| POST | `/package/get-packages` | Get all packages |
| POST | `/package/get-package` | Get single package |
| POST | `/package/purchase-package` | Purchase package |
| POST | `/package/get-all-purchase` | Get all purchases |
| POST | `/package/get-all-purchase-partner-wise` | Get purchases by partner |
| POST | `/package/get-all-topup` | Get all topups |
| POST | `/package/topup` | Topup account |
| POST | `/package/create-package-items` | Create package items |
| POST | `/package/get-packageitems` | Get package items |
| POST | `/package/getPurchaseForPartner` | Get purchases for partner |
| POST | `/package/update-onselect-Priority` | Update priority |
| POST | `/package/messages` | Package messages |
| POST | `/package/setup-postpaid-credit` | Setup postpaid credit |
| POST | `/package/update-postpaid-credit` | Update postpaid credit |

### Controller: PartnerController
**File:** `freeswitch/controller/PartnerController.java`
**Base path:** `/partner`

| Method | Path | Description |
|---|---|---|
| POST | `/partner/get-partners` | Get all partners (paginated) |
| POST | `/partner/get-typed-partners` | Get partners by type |
| POST | `/partner/get-partner` | Get single partner |
| POST | `/partner/create-partner` | Create partner |
| POST | `/partner/delete-partner` | Delete partner |
| POST | `/partner/update-partner` | Update partner |
| POST | `/partner/get-partner/{name}` | Get partner by name |

### Controller: PartnerDetailsController
**File:** `freeswitch/controller/PartnerDetailsController.java`
**Base path:** `/partner`

| Method | Path | Description |
|---|---|---|
| POST | `/partner/partner-documents` | Upload partner documents (multipart) |
| POST | `/partner/get-partner-extra` | Get partner extra info |
| POST | `/partner/get-partner-document` | Get partner document |

### Controller: PartnerPrefixController
**File:** `freeswitch/controller/PartnerPrefixController.java`
**Base path:** _(none)_

| Method | Path | Description |
|---|---|---|
| POST | `/get-partner-prefix` | Get partner prefix |
| POST | `/get-partner-prefixes` | Get all partner prefixes |
| POST | `/delete-partner-prefix` | Delete partner prefix |
| POST | `/create-partner-prefix` | Create partner prefix |
| POST | `/update-partner-prefix` | Update partner prefix |
| POST | `/get-partner-prefix-by-id-partner` | Get prefix by partner ID |
| POST | `/get-partner-prefix-by-email` | Get prefix by email |

### Controller: QosController
**File:** `freeswitch/controller/QosController.java`
**Base path:** `/qos-reports`

| Method | Path | Description |
|---|---|---|
| POST | `/qos-reports/cssr` | CSSR (Call Setup Success Rate) report |
| POST | `/qos-reports/call-drop` | Call drop report |
| POST | `/qos-reports/cssr/export-excel` | Export CSSR as Excel |
| POST | `/qos-reports/call-drop/export-excel` | Export call drop as Excel |

### Controller: RateController
**File:** `freeswitch/controller/RateController.java`
**Base path:** _(none)_
**Note:** Uses custom `@ApiController` annotation (alias for `@RestController`)

| Method | Path | Description |
|---|---|---|
| POST | `/rate-task` | Get all rate tasks |
| POST | `/new-task` | Create new rate task |

### Controller: RecordingController
**File:** `freeswitch/controller/RecordingController.java`
**Base path:** `/api/v1/recordings`
**Manages:** FusionPBX recordings (PostgreSQL)

| Method | Path | Description |
|---|---|---|
| POST | `/api/v1/recordings/create` | Create recording |
| POST | `/api/v1/recordings/list` | List all recordings |
| POST | `/api/v1/recordings/list-by-domain` | List by domain |
| POST | `/api/v1/recordings/get-by-uuid` | Get by UUID |
| POST | `/api/v1/recordings/get-by-name` | Get by name |
| POST | `/api/v1/recordings/update` | Update recording |
| POST | `/api/v1/recordings/delete` | Delete recording |
| POST | `/api/v1/recordings/download` | Download recording |

### Controller: RegistrationController
**File:** `freeswitch/controller/RegistrationController.java`
**Base path:** `/api/v1/registrations`
**Manages:** SIP registrations via FreeSWITCH ESL

| Method | Path | Description |
|---|---|---|
| POST | `/api/v1/registrations/list-by-domain` | List registrations by domain |
| POST | `/api/v1/registrations/list-all` | List all registrations |
| POST | `/api/v1/registrations/count` | Count registrations |
| POST | `/api/v1/registrations/unregister` | Force unregister |
| POST | `/api/v1/registrations/refresh` | Refresh registrations |

### Controller: ResellerDatabaseController
**File:** `freeswitch/controller/ResellerDatabaseController.java`
**Base path:** `/api/databases`

| Method | Path | Description |
|---|---|---|
| GET | `/api/databases` | List reseller databases |

### Controller: RetailPartnerController
**File:** `freeswitch/controller/RetailPartnerController.java`
**Base path:** _(none)_

| Method | Path | Description |
|---|---|---|
| POST | `/create-retail-partner` | Create retail partner |
| POST | `/create-retail-partner-from-file` | Create from file upload |
| POST | `/delete-retail-partner` | Delete retail partner |
| POST | `/get-retail-partner` | Get single retail partner |
| POST | `/get-retail-partners` | Get all retail partners |
| POST | `/update-retail-partner` | Update retail partner |
| POST | `/get-retail-partner/{idPartner}` | Get retail partner by partner ID |

### Controller: RingGroupController
**File:** `freeswitch/controller/RingGroupController.java`
**Base path:** `/api/v1/ring-groups`

| Method | Path | Description |
|---|---|---|
| POST | `/api/v1/ring-groups/list` | List ring groups |
| POST | `/api/v1/ring-groups/list-by-domain` | List by domain |
| POST | `/api/v1/ring-groups/get-by-uuid` | Get by UUID |
| POST | `/api/v1/ring-groups/create` | Create ring group |
| POST | `/api/v1/ring-groups/update` | Update ring group |
| POST | `/api/v1/ring-groups/delete` | Delete ring group |
| POST | `/api/v1/ring-groups/toggle` | Toggle ring group |
| POST | `/api/v1/ring-groups/add-destination` | Add destination to group |
| POST | `/api/v1/ring-groups/delete-destination` | Delete destination from group |

### Controller: RouteController
**File:** `freeswitch/controller/RouteController.java`
**Base path:** `/route`

| Method | Path | Description |
|---|---|---|
| POST | `/route/delete-route` | Delete route |
| POST | `/route/get-route` | Get single route |
| POST | `/route/get-routes` | Get all routes (paginated) |
| POST | `/route/create-route` | Create route |
| POST | `/route/update-route` | Update route |
| POST | `/route/get-route-by-id-partner` | Get routes by partner ID |
| POST | `/route/add-route-meta-data` | Add route metadata |

### Controller: SipCaptureCallController
**File:** `freeswitch/controller/SipCaptureCallController.java`
**Base path:** `/sip`

| Method | Path | Description |
|---|---|---|
| POST | `/sip/calls` | List SIP captured calls |
| POST | `/sip/drop` | Drop SIP call |

### Controller: SipOptionsMonitorController
**File:** `freeswitch/controller/SipOptionsMonitorController.java`
**Base path:** `/sip-options-monitor`

| Method | Path | Description |
|---|---|---|
| POST | `/sip-options-monitor/status` | Get SIP OPTIONS monitor status |
| GET | `/sip-options-monitor/configs` | Get SIP OPTIONS monitor configs |

### Controller: SmsSenderController
**File:** `freeswitch/controller/SmsSenderController.java`
**Base path:** _(none)_

| Method | Path | Description |
|---|---|---|
| POST | `/sendSmsFromExcel` | Send SMS from Excel file |

### Controller: SofiaController
**File:** `freeswitch/controller/SofiaController.java`
**Base path:** `/api/sofia`

| Method | Path | Description |
|---|---|---|
| POST | `/api/sofia/registrations` | List Sofia registrations |
| POST | `/api/sofia/partner-wise-registrations` | Partner-wise registrations |

### Controller: SummuryReportController
**File:** `freeswitch/controller/SummuryReportController.java`
**Base path:** `/api/summry-reports`

| Method | Path | Description |
|---|---|---|
| POST | `/api/summry-reports/dom-out-iptsp` | Domestic outbound IPTSP report |
| POST | `/api/summry-reports/dom-in-iptsp` | Domestic inbound IPTSP report |

### Controller: UserController
**File:** `freeswitch/controller/UserController.java`
**Base path:** `/api/v1/users`
**Manages:** FusionPBX users (PostgreSQL)

| Method | Path | Description |
|---|---|---|
| POST | `/api/v1/users/list` | List users |
| POST | `/api/v1/users/details` | Get user details |
| POST | `/api/v1/users/create` | Create user |
| POST | `/api/v1/users/update` | Update user |
| POST | `/api/v1/users/delete` | Delete user |
| GET | `/api/v1/users/groups` | List user groups |

### Controller: VExtensionController
**File:** `freeswitch/controller/VExtensionController.java`
**Base path:** `/vextension`

| Method | Path | Description |
|---|---|---|
| POST | `/vextension/get-vextensions` | Get virtual extensions |

### Controller: VXmlCdrController
**File:** `freeswitch/controller/VXmlCdrController.java`
**Base path:** _(none)_
**Manages:** XML CDR records, call history

| Method | Path | Description |
|---|---|---|
| POST | `/get-call-history` | Get call history |
| POST | `/admin/getCallHistory` | Get call history (admin) |
| POST | `/user/getCallHistory` | Get call history (user) |
| POST | `/user/downloadCallHistory` | Download call history (user, octet-stream) |
| POST | `/admin/downloadCallHistory` | Download call history (admin, octet-stream) |

### Disabled Controllers (commented out)

- **DigitCutController** (`/api/cuts`) -- digit cut CRUD, commented out
- **PrepaidBalanceController** (`/api/balance`) -- prepaid balance update/insert, commented out

---

### Database Tables (MySQL -- `l3ventures_sms`)

| Table | Entity | Description |
|---|---|---|
| `partner` | Partner | Client/carrier entities |
| `partner_extra` | PartnerExtra | Additional partner info |
| `partner_documents` | PartnerDocument | Uploaded documents |
| `partnerdoctype` | PartnerDoctype | Document types |
| `partnerprefix` | PartnerPrefix | Partner dialing prefixes |
| `retailpartner` | RetailPartner | Retail sub-partners |
| `route` | Route | Voice routing rules |
| `route_metadata` | RouteMetaData | Route extra attributes |
| `dialplan` | Dialplan | Dialplan definitions |
| `dialplan_prefix` | DialplanPrefix | Dialplan prefixes |
| `dialplan_mapping` | DialplanMapping | Dialplan-to-route mapping |
| `dialplanroute` | DialplanRoute | Dialplan routes |
| `call_src` | CallSrc | Call sources |
| `did_pool` | DidPool | DID number pools |
| `did_number` | DidNumber | Individual DID numbers |
| `did_assignment` | DidAssignment | DID-to-partner assignments |
| `did_assignment_history` | DidAssignmentHistory | Assignment audit trail |
| `cdr` | Cdr | Call Detail Records |
| `cdr_state` | CdrState | CDR state tracking |
| `account` | Account | Billing accounts |
| `acc_reserve` | AccountReserve | Account reservations |
| `package` | Package | Service packages |
| `packageitem` | PackageItem | Package line items |
| `packagepurchase` | PackagePurchase | Purchase records |
| `packageaccount` | PackageAccount | Package accounts |
| `packageaccountreserve` | PackageAccountReserve | Package account reserves |
| `contacts` | Contact | Address book contacts |
| `device_token` | DeviceToken | Push notification tokens |
| `audit_log` | AuditLog | Audit trail |
| `concurentchanneluse` | ConcurentChannelUse | Concurrent channel usage |
| `digit_filter_plan` | DigitFilterPlan | Digit filter plans |
| `digit_filter_rule` | DigitFilterRule | Digit filter rules |
| `dfr_action` | DFRAction | Digit filter rule actions |
| `dfr_action_add_prefix` | ActionAddPrefix | Add-prefix action |
| `dfr_action_allow_deny` | ActionAllowDeny | Allow/deny action |
| `dfr_action_cut_replace` | ActionCutReplace | Cut/replace action |
| `sip_capture_call` | SipCaptureCall | SIP packet capture |
| `tb_mnp` | MnpEntity | MNP (Mobile Number Portability) |
| `notification_log` | NotificationLog | Notification log |
| `auth_user` | AuthUser | Auth users (IP auth) |
| `autoincrementcounter` | AutoIncrementCounter | Auto-increment counter |
| `rateplan` | RatePlan | Rate plans |
| `rate` | Rate | Rates |
| `rateassign` | RateAssign | Rate assignments |
| `ratetask` | RateTask | Rate tasks |
| `ratetaskreference` | RateTaskReference | Rate task references |
| `ratetaskassign` | RateTaskAssign | Rate task assignments |
| `ratetaskassignreference` | RateTaskAssignReference | Rate task assign references |
| `rateplanassignmenttuple` | RatePlanAssignmentTuple | Rate plan assignment tuples |
| `voice_broadcasting_campaign` | VoiceBroadcastingCampaign | Voice broadcast campaigns |
| `user` | User | Dashboard users |
| `country` | Country | Country reference |
| `timezone` | Timezone | Timezone reference |
| `uom` | UOM | Unit of measure |
| `enumbillingspan` | EnumBillingSpan | Billing span enum |
| `enumservicecategory` | EnumServiceCategory | Service category enum |

### Database Tables (PostgreSQL -- FusionPBX `fusionpbx`)

| Table | Entity | Description |
|---|---|---|
| `v_domains` | VDomain | PBX domains/tenants |
| `v_extensions` | VExtension | PBX extensions |
| `v_gateways` | VGateway | PBX SIP gateways |
| `v_users` | VUser | PBX users |
| `v_user_groups` | VUserGroup | PBX user groups |
| `v_groups` | VGroup | PBX groups |
| `v_dialplans` | VDialplan | PBX dialplans |
| `v_dialplan_details` | VDialplanDetail | PBX dialplan details |
| `v_destinations` | VDestination | PBX destinations |
| `v_conferences` | VConference | PBX conference rooms |
| `v_ivr_menus` | VIvrMenu | PBX IVR menus |
| `v_ivr_menu_options` | VIvrMenuOption | PBX IVR menu options |
| `v_recordings` | VRecording | PBX recordings |
| `v_xml_cdr` | VXmlCdr | PBX XML CDRs |

---

## SMSREST (port 6080, service name: `smsrest`)

SMS management backend for campaigns, templates, rates, policies, and SMS sending.

### Controller: ActiveHoursController
**Base path:** `/api/activeHours`

| Method | Path | Description |
|---|---|---|
| POST | `/api/activeHours/create` | Create active hours rule |
| POST | `/api/activeHours/getAll` | Get all active hours |
| POST | `/api/activeHours/getById` | Get by ID |
| POST | `/api/activeHours/update` | Update active hours |
| POST | `/api/activeHours/delete` | Delete active hours |

### Controller: ApprovedSmsTemplateController
**Base path:** `/approved-sms-template`

| Method | Path | Description |
|---|---|---|
| POST | `/approved-sms-template/create` | Create approved template |
| POST | `/approved-sms-template/update` | Update approved template |
| POST | `/approved-sms-template/delete` | Delete approved template |
| POST | `/approved-sms-template/get-by-id` | Get by ID |
| POST | `/approved-sms-template/get-all` | Get all templates |
| POST | `/approved-sms-template/get-by-partner` | Get by partner |

### Controller: CampaignController
**Base path:** `/campaign`

| Method | Path | Description |
|---|---|---|
| POST | `/campaign/get-campaigns` | Get campaigns (admin, paginated) |
| POST | `/campaign/client/get-campaigns` | Get campaigns (client, paginated) |
| POST | `/campaign/save-campaign` | Create/save campaign |
| POST | `/campaign/disableCampaign` | Disable campaign |
| POST | `/campaign/enableCampaign` | Enable campaign |
| POST | `/campaign/get-campaign-tasks-by-campaign-name` | Get tasks by campaign name |
| POST | `/campaign/get-campaign-statistics` | Get campaign statistics |
| POST | `/campaign/get-campaign-by-id` | Get campaign by ID |

### Controller: CampaignTaskController
**Base path:** `/campaignTask`

| Method | Path | Description |
|---|---|---|
| POST | `/campaignTask/get-campaign-tasks` | Get campaign tasks (paginated) |
| POST | `/campaignTask/get-campaign-tasks-by-campaignId` | Get tasks by campaign ID |
| POST | `/campaignTask/filter` | Filter campaign tasks |
| POST | `/campaignTask/export` | Export campaign tasks |

### Controller: ContactGroupController
**Base path:** `/contact-group`

| Method | Path | Description |
|---|---|---|
| POST | `/contact-group/create` | Create contact group |
| POST | `/contact-group/update` | Update contact group |
| POST | `/contact-group/delete` | Delete contact group |
| POST | `/contact-group/getAll` | Get all groups |
| POST | `/contact-group/getByPartner` | Get groups by partner |

### Controller: ContactItemController
**Base path:** `/contact-item`

| Method | Path | Description |
|---|---|---|
| POST | `/contact-item/create` | Create contact item |
| POST | `/contact-item/update` | Update contact item |
| POST | `/contact-item/delete` | Delete contact item |
| POST | `/contact-item/getAll` | Get all items |
| POST | `/contact-item/getContactItemByGroupId` | Get items by group ID |
| POST | `/contact-item/uploadContactItems` | Upload contact items (bulk) |

### Controller: DashboardDataController
**Base path:** _(none)_

| Method | Path | Description |
|---|---|---|
| POST | `/get-intervaled-sms-data` | Get interval SMS data (admin) |
| POST | `/user/get-intervaled-sms-data` | Get interval SMS data (user) |

### Controller: DidNumberController
**Base path:** `/did-number`

| Method | Path | Description |
|---|---|---|
| POST | `/did-number/create` | Create DID number |
| POST | `/did-number/create-from-excel` | Create from Excel |
| POST | `/did-number/update-attributes` | Update DID attributes |
| POST | `/did-number/delete-attributes` | Delete DID attributes |
| POST | `/did-number/delete-single-attribute` | Delete single attribute |
| POST | `/did-number/get-all` | Get all DID numbers |
| POST | `/did-number/get-attributes` | Get DID attributes |

### Controller: DndGroupController
**Base path:** `/dnd-group`

| Method | Path | Description |
|---|---|---|
| POST | `/dnd-group/create` | Create DND group |
| POST | `/dnd-group/update` | Update DND group |
| POST | `/dnd-group/delete` | Delete DND group |
| POST | `/dnd-group/getAll` | Get all DND groups |

### Controller: DndListController
**Base path:** `/dnd-list`

| Method | Path | Description |
|---|---|---|
| POST | `/dnd-list/create` | Create DND list entry |
| POST | `/dnd-list/update` | Update DND list entry |
| POST | `/dnd-list/delete` | Delete DND list entry |
| POST | `/dnd-list/getAll` | Get all DND list entries |
| POST | `/dnd-list/getByGroup` | Get DND list by group |

### Controller: EnumJobStatusController
**Base path:** `/enum-job-status`

| Method | Path | Description |
|---|---|---|
| POST | `/enum-job-status/findAllStatus` | List all job statuses |
| POST | `/enum-job-status/findStatusById` | Get status by ID |

### Controller: EnumRouteTypeController
**Base path:** `/api/route-type`

| Method | Path | Description |
|---|---|---|
| POST | `/api/route-type/dropdown` | Get route type dropdown |

### Controller: EnumSmsErrorController
**Base path:** `/api/smserrors`

| Method | Path | Description |
|---|---|---|
| POST | `/api/smserrors/allsmserrors` | List all SMS errors |
| GET | `/api/smserrors/{id}` | Get SMS error by ID |
| POST | `/api/smserrors` | Create SMS error |
| POST | `/api/smserrors/{id}` | Update SMS error |
| DELETE | `/api/smserrors/{id}` | Delete SMS error |

### Controller: ForbiddenGroupController
**Base path:** `/forbidden-group`

| Method | Path | Description |
|---|---|---|
| POST | `/forbidden-group/create` | Create forbidden group |
| POST | `/forbidden-group/update` | Update forbidden group |
| POST | `/forbidden-group/delete` | Delete forbidden group |
| POST | `/forbidden-group/getAll` | Get all groups |

### Controller: ForbiddenWordListController
**Base path:** `/forbidden-word-list`

| Method | Path | Description |
|---|---|---|
| POST | `/forbidden-word-list/create` | Create forbidden word |
| POST | `/forbidden-word-list/update` | Update forbidden word |
| POST | `/forbidden-word-list/delete` | Delete forbidden word |
| POST | `/forbidden-word-list/getAll` | Get all forbidden words |
| POST | `/forbidden-word-list/getForbiddenWordByGroupId` | Get by group ID |

### Controller: JsonBillingRuleController
**Base path:** `/api/jsonbillingrules`

| Method | Path | Description |
|---|---|---|
| POST | `/api/jsonbillingrules/getAllRules` | Get all billing rules |
| POST | `/api/jsonbillingrules/{id}` | Get billing rule by ID |

### Controller: PolicyController
**Base path:** `/api/policies`

| Method | Path | Description |
|---|---|---|
| POST | `/api/policies/create` | Create SMS policy |
| POST | `/api/policies/getAll` | Get all policies |
| POST | `/api/policies/getById` | Get policy by ID |
| POST | `/api/policies/update` | Update policy |
| POST | `/api/policies/delete` | Delete policy |

### Controller: RateController
**Base path:** `/rate`

| Method | Path | Description |
|---|---|---|
| POST | `/rate/getAllRate` | Get all SMS rates |
| POST | `/rate/getRateByRateplanId` | Get rates by rate plan ID |
| POST | `/rate/getRateByIdPartner` | Get rates by partner ID |
| POST | `/rate/search` | Search rates |

### Controller: RatePlanController
**Base path:** `/api/rate-plan`

| Method | Path | Description |
|---|---|---|
| POST | `/api/rate-plan/create` | Create rate plan |
| POST | `/api/rate-plan/getAll` | Get all rate plans |
| POST | `/api/rate-plan/getAllRatePlanById` | Get rate plan by ID |
| POST | `/api/rate-plan/search` | Search rate plans |
| POST | `/api/rate-plan/update` | Update rate plan |
| POST | `/api/rate-plan/deleteRatePlan` | Delete rate plan |

### Controller: RatePlanAssignmentController
**Base path:** `/api/rateplan-assignment`

| Method | Path | Description |
|---|---|---|
| POST | `/api/rateplan-assignment/assign` | Assign rate plan |
| POST | `/api/rateplan-assignment/allRatePlanAssign` | List all assignments |
| POST | `/api/rateplan-assignment/deletePlanAssign` | Delete assignment |

### Controller: RatePlanAssignmentTupleController
**Base path:** `/api/rateplanassignmenttuples`

| Method | Path | Description |
|---|---|---|
| POST | `/api/rateplanassignmenttuples/existing` | Get existing tuples |
| POST | `/api/rateplanassignmenttuples/create` | Create tuple |

### Controller: RatePlanDropDownController
**Base path:** `/api/drop-down`

| Method | Path | Description |
|---|---|---|
| POST | `/api/drop-down/getServiceFamily` | Get service family dropdown |
| POST | `/api/drop-down/getServiceGroup` | Get service group dropdown |
| POST | `/api/drop-down/getPartnerType` | Get partner type dropdown |
| POST | `/api/drop-down/getTimeZone` | Get timezone dropdown |
| POST | `/api/drop-down/getAllUom` | Get UOM dropdown |
| POST | `/api/drop-down/getAllBillingsSpan` | Get billing span dropdown |
| POST | `/api/drop-down/getAllServiceCategory` | Get service category dropdown |
| POST | `/api/drop-down/getAllServiceSubCategory` | Get service sub-category dropdown |
| POST | `/api/drop-down/getAllCountryCode` | Get country code dropdown |

### Controller: RateTaskController
**Base path:** `/api/ratetask`

| Method | Path | Description |
|---|---|---|
| POST | `/api/ratetask/createRateTask` | Create rate task |
| POST | `/api/ratetask/finalizeRateTaskCommit` | Finalize/commit rate task |
| POST | `/api/ratetask/insertDirect` | Insert rate directly |
| POST | `/api/ratetask/insertBulkDirect` | Bulk insert rates |
| POST | `/api/ratetask/getAllRateTask` | Get all rate tasks |
| POST | `/api/ratetask/getAllRateTaskByTaskRef` | Get rate tasks by reference |

### Controller: RateTaskReferenceController
**Base path:** `/api/ratetaskreference`

| Method | Path | Description |
|---|---|---|
| POST | `/api/ratetaskreference/createReference` | Create reference |
| POST | `/api/ratetaskreference/updateReference` | Update reference |
| POST | `/api/ratetaskreference/getRateReferenceByRatePlanId` | Get by rate plan ID |

### Controller: ReportController
**Base path:** `/reports`

| Method | Path | Description |
|---|---|---|
| POST | `/reports/sms` | Generate SMS report |

### Controller: RetryCauseCodeController
**Base path:** `/api/retry-cause-codes`

| Method | Path | Description |
|---|---|---|
| POST | `/api/retry-cause-codes/create` | Create retry cause code |
| POST | `/api/retry-cause-codes/update` | Update retry cause code |
| POST | `/api/retry-cause-codes` | List all retry cause codes |

### Controller: RetryIntervalController
**Base path:** `/api/retry-intervals`

| Method | Path | Description |
|---|---|---|
| POST | `/api/retry-intervals/create` | Create retry interval |
| POST | `/api/retry-intervals` | List retry intervals |
| POST | `/api/retry-intervals/allRetryIntervals` | List all retry intervals |

### Controller: RetryRuleController
**Base path:** `/api/retryRules`

| Method | Path | Description |
|---|---|---|
| POST | `/api/retryRules/create` | Create retry rule |
| POST | `/api/retryRules/getAll` | Get all retry rules |
| POST | `/api/retryRules/getById` | Get by ID |
| POST | `/api/retryRules/getByPolicyId` | Get by policy ID |
| POST | `/api/retryRules/update` | Update retry rule |
| POST | `/api/retryRules/delete` | Delete retry rule |

### Controller: SchedulePolicyController
**Base path:** `/api/schedulePolicies`

| Method | Path | Description |
|---|---|---|
| POST | `/api/schedulePolicies/create` | Create schedule policy |
| POST | `/api/schedulePolicies/getAll` | Get all policies |
| POST | `/api/schedulePolicies/getById` | Get by ID |
| POST | `/api/schedulePolicies/update` | Update policy |
| POST | `/api/schedulePolicies/delete` | Delete policy |

### Controller: SendSmsTaskController
**Base path:** `/SmsTask`

| Method | Path | Description |
|---|---|---|
| POST | `/SmsTask/sendSms` | Send SMS (campaign task) |
| POST | `/SmsTask/sendSmsAggregator` | Send SMS via aggregator |
| POST | `/SmsTask/dynamicSendSms` | Send SMS with dynamic template (multipart) |

### Controller: SmsClientDashboardController
**Base path:** `/api/client/dashboard`

| Method | Path | Description |
|---|---|---|
| POST | `/api/client/dashboard/get-top-campaign-task` | Get top campaign tasks |
| POST | `/api/client/dashboard/incoming/sms-stats` | Incoming SMS stats |
| POST | `/api/client/dashboard/outgoing/sms-stats` | Outgoing SMS stats |

### Controller: SmsControllerGenericDTO
**Base path:** `/api/dto`

| Method | Path | Description |
|---|---|---|
| POST | `/api/dto/campaigns/tasks` | Get campaign tasks (generic DTO) |
| POST | `/api/dto/campaigns/{date}/tasks` | Get tasks by date |

### Controller: SmsDashboardController
**Base path:** `/api/admin/dashboard`

| Method | Path | Description |
|---|---|---|
| POST | `/api/admin/dashboard/sms-and-campaign-summary` | SMS and campaign summary |
| POST | `/api/admin/dashboard/get-top-5-partner-sms-count` | Top 5 partner SMS counts |
| POST | `/api/admin/dashboard/get-top-campaign-task` | Top campaign tasks |
| POST | `/api/admin/dashboard/incoming/sms-stats` | Incoming SMS stats |
| POST | `/api/admin/dashboard/outgoing/sms-stats` | Outgoing SMS stats |

### Controller: SmsGatewayController
**Base path:** `/sms`

| Method | Path | Description |
|---|---|---|
| POST | `/sms/send` | Send single SMS via gateway |

### Controller: SmsQueueController
**Base path:** `/api/sms-queue`

| Method | Path | Description |
|---|---|---|
| POST | `/api/sms-queue/getAll` | Get all SMS queues |
| POST | `/api/sms-queue/getSmsQueueById` | Get queue by ID |
| POST | `/api/sms-queue/create` | Create SMS queue |
| POST | `/api/sms-queue/update` | Update SMS queue |
| POST | `/api/sms-queue/delete` | Delete SMS queue |

### Controller: SmsTaskInfozillion (v2 API)
**Base path:** `/api/v2`

| Method | Path | Description |
|---|---|---|
| POST | `/api/v2/promo/sendSms` | Send promo SMS (v2) |
| POST | `/api/v2/trans/sendSms` | Send transactional SMS (v2) |
| GET | `/api/v2/promo/sendSmsViaGet` | Send promo SMS via GET |

### Controller: SmsTemplateController
**Base path:** `/sms-template`

| Method | Path | Description |
|---|---|---|
| POST | `/sms-template/create` | Create SMS template |
| POST | `/sms-template/update` | Update SMS template |
| POST | `/sms-template/delete` | Delete SMS template |
| POST | `/sms-template/getById` | Get template by ID |
| POST | `/sms-template/getAll` | Get all templates |
| POST | `/sms-template/getByPartner` | Get by partner |
| POST | `/sms-template/getByPartnerPaginated` | Get by partner (paginated) |
| POST | `/sms-template/getGlobalTemplates` | Get global templates |
| POST | `/sms-template/search` | Search templates |

### Controller: ThrottlingRuleController
**Base path:** `/api/throttlingRules`

| Method | Path | Description |
|---|---|---|
| POST | `/api/throttlingRules/create` | Create throttling rule |
| POST | `/api/throttlingRules/getAll` | Get all rules |
| POST | `/api/throttlingRules/getById` | Get by ID |
| POST | `/api/throttlingRules/getByPolicyId` | Get by policy ID |
| POST | `/api/throttlingRules/update` | Update rule |
| POST | `/api/throttlingRules/delete` | Delete rule |

### Controller: TimeBandController
**Base path:** `/api/timeBands`

| Method | Path | Description |
|---|---|---|
| POST | `/api/timeBands/create` | Create time band |
| POST | `/api/timeBands/getAll` | Get all time bands |
| POST | `/api/timeBands/getById` | Get by ID |
| POST | `/api/timeBands` | List time bands |
| POST | `/api/timeBands/update` | Update time band |
| POST | `/api/timeBands/delete` | Delete time band |

### Controller: TopicController
**Base path:** _(none)_

| Method | Path | Description |
|---|---|---|
| POST | `/hello` | Health check |
| POST | `/create-topic` | Create Kafka topic |
| POST | `/delete-topic` | Delete Kafka topic |
| POST | `/save-sms` | Save SMS record |

### Controller: VoiceBroadcastController
**Base path:** `/voice-broadcast`

| Method | Path | Description |
|---|---|---|
| POST | `/voice-broadcast/get-voice-broadcasts` | Get broadcasts (admin, paginated) |
| POST | `/voice-broadcast/client/get-voice-broadcasts` | Get broadcasts (client, paginated) |
| POST | `/voice-broadcast/save-voice-broadcast` | Save voice broadcast (multipart) |
| POST | `/voice-broadcast/disable-voice-broadcast` | Disable broadcast |
| POST | `/voice-broadcast/enable-voice-broadcast` | Enable broadcast |
| POST | `/voice-broadcast/get-voice-broadcast-by-id` | Get by ID |
| GET | `/voice-broadcast/download-audio/{campaignId}` | Download broadcast audio |
| POST | `/voice-broadcast/get-task-state-counts` | Get task state counts |

---

## AUTHENTICATION (port 8080, service name: `AUTHENTICATION`)

JWT-based authentication, authorization, RBAC, and user management.

### Controller: AuthController
**Base path:** `/auth`

| Method | Path | Description |
|---|---|---|
| POST | `/auth/createUser` | Create user |
| POST | `/auth/login` | User login (returns JWT) |
| POST | `/auth/webrtc/login` | WebRTC login |

### Controller: AdminController
**Base path:** `/admin`

| Method | Path | Description |
|---|---|---|
| POST | `/admin/create-admin-user` | Create admin user |
| POST | `/admin/manage-user-roles` | Manage user roles |

### Controller: SecurityUtilsController
**Base path:** `/auth`

| Method | Path | Description |
|---|---|---|
| POST | `/auth/validateAccess` | Validate JWT token (used by gateway) |

### Controller: PermissionCrudController
**Base path:** `/auth`

| Method | Path | Description |
|---|---|---|
| POST | `/auth/createPermission` | Create permission |
| POST | `/auth/deletePermission/{id}` | Delete permission |
| POST | `/auth/getPermissions` | Get all permissions |
| POST | `/auth/getPermission/{id}` | Get permission by ID |
| POST | `/auth/editPermission` | Edit permission |
| POST | `/auth/get-roles-with-permission-id/{permissionId}` | Get roles by permission ID |

### Controller: RoleCrudController
**Base path:** `/auth`

| Method | Path | Description |
|---|---|---|
| POST | `/auth/createRole` | Create role |
| POST | `/auth/getRoles` | Get all roles |
| POST | `/auth/getRole/{id}` | Get role by ID |
| POST | `/auth/deleteRole/{id}` | Delete role |
| POST | `/auth/setRolePermissions/{roleId}` | Set permissions for role |
| POST | `/auth/get-users-with-role-id/{roleId}` | Get users by role ID |

### Controller: PbxMenuPermissionController
**Base path:** `/permissions`

| Method | Path | Description |
|---|---|---|
| GET | `/permissions/available-menus` | List available PBX menus |
| POST | `/permissions/get-user-permissions` | Get user's menu permissions |
| POST | `/permissions/save` | Save menu permissions |
| POST | `/permissions/create-pbx-user` | Create PBX user with permissions |
| POST | `/permissions/check` | Check permission |
| POST | `/permissions/delete` | Delete permission assignment |

### Controller: UserController
**Base path:** _(none)_

| Method | Path | Description |
|---|---|---|
| POST | `/deleteUser` | Delete user |
| POST | `/editUser` | Edit user |
| POST | `/getUser` | Get single user |
| POST | `/getUserByEmail` | Get user by email |
| POST | `/getUserByIdPartner` | Get users by partner ID |
| POST | `/getUsers` | Get all users |
| POST | `/logout` | User logout |

### Controller: InternalServiceUtils
**Base path:** `/system-api`

| Method | Path | Description |
|---|---|---|
| POST | `/system-api/get-reseller-database-with-token` | Get reseller DB by JWT token |
| POST | `/system-api/add-reseller-database` | Add reseller database |
| POST | `/system-api/get-reseller-database-with-partner-id` | Get reseller DB by partner ID |
| POST | `/system-api/get-username-by-phone-no` | Get username by phone number |
| POST | `/system-api/get-idpartner-from-jwt` | Extract partner ID from JWT |

### Controller: WebTokenController
**Base path:** `/token`

| Method | Path | Description |
|---|---|---|
| POST | `/token/create-api-key` | Create API key |
| POST | `/token/delete-api-key` | Delete API key |
| POST | `/token/get-active-api-keys-for-user` | List active API keys for user |

---

## Summary Statistics

| Service | Controllers | Endpoints | Primary Database |
|---|---|---|---|
| FreeSwitchREST | 47 active | ~280 | MySQL (`l3ventures_sms`) + PostgreSQL (`fusionpbx`) |
| smsrest | 41 | ~140 | MySQL (via smsrest DB) |
| AUTHENTICATION | 10 | ~30 | MySQL (via Security DB) |
| **Total** | **98** | **~450** | |
