# Voice/Softswitch Features -- RTC-Manager Backend

This document catalogs all voice/call/FreeSWITCH related features in the RTC-Manager backend, focused on the CCL (Cosmocom Limited) tenant deployment.

---

## FreeSwitchREST Controllers (Voice)

FreeSwitchREST is a Spring Boot service running on port **5071** (Eureka name: `FreeswitchRest`).
All endpoints are proxied via the API-Gateway on port **8001** under the `/FREESWITCHREST/` prefix.

---

### ActiveCallController
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/ActiveCallController.java`
- **Base:** `/api/v1/active-calls`
- POST `/list` -- List all currently active calls/channels on FreeSWITCH (ESL `show channels`)
- POST `/details` -- Get detailed info about a specific active call by UUID
- POST `/hangup` -- Terminate an active call by UUID (ESL uuid_kill)
- POST `/transfer` -- Transfer an active call to another destination
- POST `/hold` -- Place/remove an active call from hold (hold/unhold/toggle)
- POST `/mute` -- Mute/unmute agent microphone on an active call
- POST `/conference` -- Create 3-way conference by adding third party to active call

### FsController (Live Calls & Concurrent Call Monitoring)
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/FsController.java`
- **Base:** `/` (root-level paths)
- POST `/profile-path` -- Get FreeSWITCH user XML profile path
- POST `/getConcurrentCall` -- Get current concurrent call count
- POST `/get-partner-details` -- Get concurrent calls broken down by partner
- POST `/get-btcl-calls` -- Get concurrent calls for BTCL partner
- POST `/get-outgoing-calls` -- Get outgoing calls broken down by partner
- POST `/get-partner-concurrent-call-limit` -- Get partner concurrent call limits
- POST `/concurrent-call-profile` -- Get concurrent call profile data
- POST `/concurrent-call-profile-partner` -- Get concurrent call profiles for a specific partner
- POST `/concurrent-call-partner-wise` -- Get concurrent call counts per partner
- POST `/admin-live-calls-summary` -- Admin-level live call summary (total, inbound, outbound)
- POST `/user-live-calls-summary` -- User-level live call summary for a specific partner

### CdrController (CDR - Legacy MySQL)
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/CdrController.java`
- **Base:** `/api/cdr`
- POST `/cdrByFilterForAdmin` -- Get filtered CDRs for admin (paginated)
- POST `/cdrByFilterExportForAdmin` -- Export filtered CDRs as file for admin
- POST `/cdrByFilterForUser` -- Get filtered CDRs for user (paginated)
- POST `/cdrByFilterExportForUser` -- Export filtered CDRs as file for user

### CdrApiController (CDR - FusionPBX PostgreSQL)
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/CdrApiController.java`
- **Base:** `/api/v1/cdr`
- POST `/list-by-domain` -- Get CDR list by domain with filters (direction, hangupCause, caller, callee, date range, codec, duration range) and pagination
- POST `/stats` -- Get CDR statistics for a domain (total, inbound, outbound, internal, missed, answered, avgDuration)
- POST `/get-by-uuid` -- Get single CDR record by UUID
- GET `/hangup-causes` -- Get list of standard hangup cause options

### CallRecordingController
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/CallRecordingController.java`
- **Base:** `/api/recordings`
- POST `/get-recordings` -- Get filtered call recordings (caller, callee, date range, paginated)
- POST `/search-recordings` -- Full-text search recordings
- POST `/download-zip` -- Download recordings as ZIP archive
- POST `/auto-load` -- Auto-discover and load recording files into DB
- POST `/stream` -- Stream a recording by ID (audio/mp3)
- **FusionPBX PHP API endpoints (v1):**
  - POST `/v1/list` -- List all call recordings via FusionPBX REST API
  - POST `/v1/list-by-domain` -- List recordings filtered by domain
  - POST `/v1/get-by-uuid` -- Get recording details by recording UUID
  - POST `/v1/delete` -- Delete a call recording
  - POST `/v1/get-by-call-uuid` -- Get recording by call UUID (optional base64)
  - POST `/v1/get-by-sip-call-id` -- Get recording by SIP Call-ID with fallback search
  - POST `/v1/bulk-info` -- Get bulk recording info (count before download)
  - POST `/v1/bulk-download` -- Bulk download recordings as ZIP (by UUID list or date range)
  - POST `/v1/download` -- Get download info for a recording (optional base64)

### RecordingController (Audio File Management)
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/RecordingController.java`
- **Base:** `/api/v1/recordings`
- POST `/create` -- Upload a new recording (base64 audio content) via FusionPBX PHP API
- POST `/list` -- List all recordings
- POST `/list-by-domain` -- List recordings by domain
- POST `/get-by-uuid` -- Get recording by UUID
- POST `/get-by-name` -- Get recording by name and domain
- POST `/update` -- Update recording metadata
- POST `/delete` -- Delete recording via FusionPBX PHP API
- POST `/download` -- Download recording with base64 content

### RegistrationController (SIP Registrations)
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/RegistrationController.java`
- **Base:** `/api/v1/registrations`
- POST `/list-by-domain` -- Get all SIP registrations for a domain (via FusionPBX REST API)
- POST `/list-all` -- Get all SIP registrations across all domains (admin)
- POST `/count` -- Get registration count by profile/domain
- POST `/unregister` -- Force unregister/flush a SIP registration
- POST `/refresh` -- Refresh registrations (no-op, real-time fetch)

### SofiaController (Sofia SIP Status)
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/SofiaController.java`
- **Base:** `/api/sofia`
- POST `/registrations` -- Get filtered SIP registrations from sofia status
- POST `/partner-wise-registrations` -- Get SIP registrations grouped by partner

### GatewayController (SIP Trunks/Gateways)
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/GatewayController.java`
- **Base:** `/api/v1/gateways`
- POST `/list` -- List all gateways (optionally filtered by domain)
- POST `/list-by-domain` -- List gateways by domain
- POST `/details` -- Get gateway details by UUID
- POST `/get-by-uuid` -- Alias for details
- POST `/create` -- Create new SIP gateway (proxy, profile, register, callerIdInFrom, etc.)
- POST `/update` -- Update an existing gateway
- POST `/delete` -- Delete a gateway
- POST `/toggle` -- Toggle gateway enabled/disabled
- GET `/profiles` -- List available gateway profiles (internal, external, etc.)
- GET `/transports` -- List available gateway transports

### ExtensionController (FusionPBX Extensions)
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/ExtensionController.java`
- **Base:** `/api/v1/extensions`
- POST `/create` -- Create extension(s), supports range for bulk creation
- POST `/list` -- List all extensions
- POST `/list-by-domain` -- List extensions by domain
- POST `/get-by-uuid` -- Get extension by UUID
- POST `/get-by-number` -- Get extension by number and domain
- POST `/update` -- Update extension
- POST `/delete` -- Delete extension
- POST `/toggle` -- Toggle extension enabled/disabled

### VExtensionController (PostgreSQL Extension View)
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/VExtensionController.java`
- **Base:** `/vextension`
- POST `/get-vextensions` -- List FusionPBX extensions from PostgreSQL (paginated)

### IvrMenuController (IVR Menus)
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/IvrMenuController.java`
- **Base:** `/api/v1/ivr-menus`
- POST `/create` -- Create IVR menu with optional menu options
- POST `/list` -- List all IVR menus
- POST `/list-by-domain` -- List IVR menus by domain
- POST `/get-by-uuid` -- Get IVR menu by UUID
- POST `/get-by-extension` -- Get IVR menu by extension and domain
- POST `/update` -- Update IVR menu
- POST `/delete` -- Delete IVR menu and its options
- POST `/toggle` -- Toggle IVR menu enabled/disabled
- POST `/add-option` -- Add option to an IVR menu (digit, action, param)
- POST `/delete-option` -- Delete an IVR menu option

### ConferenceController
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/ConferenceController.java`
- **Base:** `/api/v1/conferences`
- POST `/create` -- Create conference via FusionPBX PHP API
- POST `/list-by-domain` -- List conferences by domain
- POST `/get-by-uuid` -- Get conference details
- POST `/update` -- Update conference
- POST `/delete` -- Delete conference
- POST `/toggle` -- Toggle conference enabled/disabled

### RingGroupController
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/RingGroupController.java`
- **Base:** `/api/v1/ring-groups`
- POST `/list` -- List all ring groups (optional domain filter)
- POST `/list-by-domain` -- List ring groups by domain
- POST `/get-by-uuid` -- Get ring group details
- POST `/create` -- Create ring group via FusionPBX PHP API
- POST `/update` -- Update ring group
- POST `/delete` -- Delete ring group
- POST `/toggle` -- Toggle ring group enabled/disabled
- POST `/add-destination` -- Add destination to ring group
- POST `/delete-destination` -- Remove destination from ring group

### InboundRouteController
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/InboundRouteController.java`
- **Base:** `/api/v1/inbound-routes`
- POST `/create` -- Create inbound route (dialplan for incoming DID routing)
- POST `/list-by-domain` -- List inbound routes by domain
- POST `/get-by-uuid` -- Get inbound route by UUID
- POST `/update` -- Update inbound route
- POST `/delete` -- Delete inbound route
- POST `/toggle` -- Toggle inbound route enabled/disabled

### OutboundRouteController
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/OutboundRouteController.java`
- **Base:** `/api/v1/outbound-routes`
- POST `/list` -- List all outbound routes (optional domain/context filter)
- POST `/details` -- Get outbound route details
- POST `/create` -- Create outbound route (dialplan_name, gateway_uuid, destination_pattern)
- POST `/update` -- Update outbound route
- POST `/delete` -- Delete outbound route
- POST `/toggle` -- Toggle outbound route enabled/disabled

### DialplanController (Voice Routing Dialplans)
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/DialplanController.java`
- **Base:** `/` (root-level paths)
- POST `/get-dialplan` -- Get dialplan by call source ID
- POST `/create-dialplan` -- Create a new dialplan
- POST `/get-dialplan-by-id` -- Get dialplan by ID
- POST `/get-dialplans` -- List all dialplans (paginated)
- POST `/update-dialplan` -- Update dialplan
- POST `/delete-dialplan` -- Delete dialplan
- POST `/add-route-to-dialplan` -- Add a route to a dialplan
- POST `/delete-dialplan-route` -- Remove a route from a dialplan

### DialPlanPrefixController
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/DialPlanPrefixController.java`
- **Base:** `/` (root-level paths)
- POST `/create-dialplan-prefix` -- Create dialplan prefix mapping
- POST `/update-dialplan-prefix` -- Update dialplan prefix
- POST `/get-dialplan-prefix` -- Get dialplan prefix by ID
- POST `/delete-dialplan-prefix` -- Delete dialplan prefix
- POST `/dialplan-prefix/exists` -- Check if a prefix exists for a call source
- POST `/get-dialplan-prefixes` -- List all dialplan prefixes (paginated)
- POST `/get-dialplan-prefixes-grouped` -- List prefixes grouped by call source

### RouteController (Voice Routes)
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/RouteController.java`
- **Base:** `/route`
- POST `/delete-route` -- Delete a voice route
- POST `/get-route` -- Get route with partner details
- POST `/get-routes` -- List all routes (paginated)
- POST `/create-route` -- Create route with route metadata
- POST `/update-route` -- Update route with metadata
- POST `/get-route-by-id-partner` -- Get routes by partner ID
- POST `/add-route-meta-data` -- Add metadata to a route

### Did__Controller (DID Management)
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/Did__Controller.java`
- **Base:** `/` (root-level paths)
- **DID Pools:**
  - POST `/get-did-pool` -- Get DID pool by ID
  - POST `/get-did-pools` -- List all DID pools (paginated)
  - POST `/update-did-pool` -- Update DID pool
  - POST `/create-did-pool` -- Create DID pool
  - POST `/delete-did-pool` -- Delete DID pool
- **DID Numbers:**
  - POST `/get-did-number` -- Get DID number by key
  - POST `/get-did-numbers` -- List all DID numbers (paginated)
  - POST `/get-unassigned-did-numbers` -- List unassigned DIDs
  - POST `/update-did-number` -- Bulk update DID numbers
  - POST `/delete-did-number` -- Delete DID number
  - POST `/create-did-number` -- Create DID number
  - POST `/create-did-number-excel` -- Bulk import DIDs from Excel
  - POST `/get-did-numbers-by-pool` -- List DIDs by pool (paginated)
- **DID Assignments:**
  - POST `/create-did-assignment` -- Assign DIDs to partner
  - POST `/update-did-assignment` -- Update DID assignment
  - POST `/delete-did-assignment` -- Delete DID assignment
  - POST `/get-did-assignment` -- Get assignment by ID
  - POST `/get-did-assignment-by-partner-id` -- List assignments by partner (paginated)
  - POST `/get-did-assignment-history-by-partner-id` -- Assignment history by partner
  - POST `/get-did-assignments` -- List all assignments (paginated)
  - POST `/create-did-assign-from-csv` -- Bulk assign DIDs from CSV
  - POST `/get-did-assignment-history` -- Full DID assignment history (paginated)

### PartnerController (Voice Partners / IPTSP Clients)
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/PartnerController.java`
- **Base:** `/partner`
- POST `/get-partners` -- List all partners (paginated)
- POST `/get-typed-partners` -- List partners by type (paginated)
- POST `/get-partner` -- Get partner by ID
- POST `/create-partner` -- Create partner (sends welcome email)
- POST `/delete-partner` -- Delete partner
- POST `/update-partner` -- Update partner
- POST `/get-partner/{name}` -- Get partner by name

### PartnerDetailsController (Partner Documents)
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/PartnerDetailsController.java`
- **Base:** `/partner`
- POST `/partner-documents` -- Upload partner documents (multipart)
- POST `/get-partner-extra` -- Get partner extra details
- POST `/get-partner-document` -- Download a partner document by type

### PartnerPrefixController
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/PartnerPrefixController.java`
- **Base:** `/` (root-level paths)
- POST `/get-partner-prefix` -- Get partner prefix by ID
- POST `/get-partner-prefixes` -- List all partner prefixes (paginated)
- POST `/delete-partner-prefix` -- Delete partner prefix
- POST `/create-partner-prefix` -- Create partner prefix
- POST `/update-partner-prefix` -- Update partner prefix
- POST `/get-partner-prefix-by-id-partner` -- List prefixes by partner
- POST `/get-partner-prefix-by-email` -- Get prefixes by partner email

### CallSrcController (Call Sources / Trunks)
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/CallSrcController.java`
- **Base:** `/` (root-level paths)
- POST `/create-callsrc` -- Create call source (trunk definition)
- POST `/get-callsrcs` -- List all call sources (paginated)
- POST `/update-callsrc` -- Update call source

### CallForwardController
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/CallForwardController.java`
- **Base:** `/api/v1/call-forward`
- POST `/list-by-domain` -- List call forward settings for all extensions in a domain
- POST `/get-by-extension` -- Get call forward settings for a specific extension
- POST `/update` -- Update call forward settings (via FusionPBX PHP API for cache clear)
- POST `/toggle-dnd` -- Toggle Do Not Disturb
- POST `/toggle-forward-all` -- Toggle Forward All

### CallBlockController
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/CallBlockController.java`
- **Base:** `/api/v1/call-block`
- POST `/list-by-domain` -- List call blocks by domain (direction, enabled filters)
- POST `/create` -- Create call block rule
- POST `/update` -- Update call block
- POST `/delete` -- Delete call block
- POST `/get-by-uuid` -- Get call block details
- POST `/toggle` -- Toggle call block enabled/disabled

### CallPermissionsController
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/CallPermissionsController.java`
- **Base:** `/api/v1/call-permissions`
- POST `/list-by-domain` -- List call permissions for all extensions in a domain
- POST `/update` -- Update call permissions for an extension

### CallbackController (Callback Queue)
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/CallbackController.java`
- **Base:** `/api/v1/callback`
- **Configuration:**
  - POST `/config/create` -- Create callback configuration
  - POST `/config/list` -- List callback configs (by domain, queue, enabled)
  - POST `/config/get` -- Get config details
  - POST `/config/update` -- Update config
  - POST `/config/delete` -- Delete config
  - POST `/config/toggle` -- Toggle config enabled/disabled
- **Queue:**
  - POST `/queue/create` -- Create callback (manual or triggered)
  - POST `/queue/list` -- List callbacks in queue
  - POST `/queue/get` -- Get callback details
  - POST `/queue/cancel` -- Cancel a callback
  - POST `/queue/retry` -- Manually retry a callback
- POST `/install` -- Auto-install callback tables

### CallCenterController (ACD Queues & Agents)
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/CallCenterController.java`
- **Base:** `/api/call-center`
- **Queues:**
  - POST `/v1/queues/list` -- List call center queues
  - POST `/v1/queues/details` -- Get queue details
  - POST `/v1/queues/status` -- Get queue real-time status
  - POST `/v1/queues/live` -- Get live queue monitoring (agents, waiting callers, stats)
  - POST `/v1/queues/create` -- Create queue (name, extension, strategy)
  - POST `/v1/queues/update` -- Update queue
  - POST `/v1/queues/delete` -- Delete queue
- **Agents:**
  - POST `/v1/agents/list` -- List agents
  - POST `/v1/agents/details` -- Get agent details
  - POST `/v1/agents/status` -- Get agent real-time status
  - POST `/v1/agents/set-status` -- Set agent status (Available, On Break, Logged Out)
  - POST `/v1/agents/set-state` -- Set agent state (Waiting, Idle)
  - POST `/v1/agents/create` -- Create agent
  - POST `/v1/agents/update` -- Update agent
  - POST `/v1/agents/delete` -- Delete agent
  - POST `/v1/agents/stats` -- Get agent performance KPIs (callsHandled, missedCalls, avgHandleTime, etc.)
- **Tiers (Agent-Queue Assignments):**
  - POST `/v1/tiers/list` -- List tiers (by domain, queue, agent)
  - POST `/v1/tiers/add` -- Add agent to queue (with level/position)
  - POST `/v1/tiers/remove` -- Remove agent from queue
- **Supervisor:**
  - POST `/v1/eavesdrop` -- Eavesdrop on an active call (spy/whisper)

### BroadcastController (Voice Broadcasting - Legacy)
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/BroadcastController.java`
- **Base:** `/api/broadcast`
- POST `/play-audio` -- Upload audio (WAV) and broadcast to list of SIP numbers via fs_cli originate

### CallBroadcastController (Voice Broadcasting - FusionPBX)
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/CallBroadcastController.java`
- **Base:** `/api/call-broadcast`
- POST `/v1/list` -- List call broadcasts
- POST `/v1/details` -- Get broadcast details
- POST `/v1/create` -- Create broadcast campaign
- POST `/v1/update` -- Update broadcast
- POST `/v1/delete` -- Delete broadcast
- POST `/v1/start` -- Start broadcast (supervisor action)
- POST `/v1/stop` -- Stop broadcast
- POST `/v1/upload-leads` -- Upload phone numbers to broadcast

### DomainController (FusionPBX Multi-Tenant Domains)
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/DomainController.java`
- **Base:** `/api/v1/domains`
- POST `/create` -- Create domain (also auto-creates route entry)
- POST `/list` -- List all domains
- POST `/get-by-uuid` -- Get domain by UUID
- POST `/get-by-name` -- Get domain by name
- POST `/update` -- Update domain
- POST `/delete` -- Delete domain
- POST `/toggle` -- Toggle domain enabled/disabled

### QosController (Quality of Service Reports)
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/QosController.java`
- **Base:** `/qos-reports`
- POST `/cssr` -- Get CSSR (Call Setup Success Rate) report by period
- POST `/call-drop` -- Get call drop rate statistics
- POST `/cssr/export-excel` -- Export CSSR report as Excel
- POST `/call-drop/export-excel` -- Export call drop report as Excel

### SipOptionsMonitorController
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/SipOptionsMonitorController.java`
- **Base:** `/sip-options-monitor`
- POST `/status` -- Check SIP OPTIONS status of all monitored gateway servers
- GET `/configs` -- Get all SIP OPTIONS monitor configurations

### DashboardControllerAdmin (Admin Dashboard)
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/DashboardControllerAdmin.java`
- **Base:** `/admin/DashBoard`
- POST `/getTotalCall` -- Total calls count for admin
- POST `/getOutgoingCall` -- Outgoing calls count
- POST `/getIncomingCall` -- Incoming calls count
- POST `/getMissedCall` -- Missed calls count
- POST `/getIntervalWiseCall` -- Interval-wise call chart data
- POST `/get-partner-balances` -- Partner package balances (paginated)
- POST `/get-partner-balances-unified` -- Unified partner balances (prepaid/postpaid)
- POST `/system-info` -- System monitoring info (CPU, memory, disk)
- POST `/top5PartnerCallCounts` -- Top 5 partners by call count
- POST `/partner/validate` -- Validate partner credentials

### DashboardControllerUser (User Dashboard)
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/DashboardControllerUser.java`
- **Base:** `/user/DashBoard`
- POST `/getTotalCall` -- Total calls for user/partner
- POST `/getOutgoingCall` -- Outgoing calls for user
- POST `/getIncomingCall` -- Incoming calls for user
- POST `/getMissedCall` -- Missed calls for user
- POST `/getIntervalWiseCall` -- Interval-wise call data for user
- POST `/getTopupBalanceForUser` -- Get wallet balance (prepaid)
- POST `/getBalance` -- Get unified balance (prepaid wallet or postpaid credit)
- POST `/processPostpaidPayment` -- Process postpaid payment
- POST `/addPostpaidUsage` -- Add usage to postpaid credit

### SummuryReportController (Summary Reports)
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/SummuryReportController.java`
- **Base:** `/api/summry-reports`
- POST `/dom-out-iptsp` -- Generate outgoing summary report (domestic out, IPTSP)
- POST `/dom-in-iptsp` -- Generate incoming summary report (domestic in, IPTSP)

### ConcurentCallAnalyticsController
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/ConcurentCallAnalyticsController.java`
- **Base:** `/analytics`
- POST `/graph-data` -- Get concurrent call graph data points
- POST `/graph-data/csv` -- Export concurrent call graph data as CSV

### MaximumConcurrentCallController
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/MaximumConcurrentCallController.java`
- **Base:** `/mcc-reports`
- POST `/max-call-periodically` -- Get max concurrent calls report by period
- POST `/export/csv` -- Export MCC report as CSV
- POST `/export/excel` -- Export MCC report as Excel

### BalanceController
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/BalanceController.java`
- **Base:** `/` (root-level paths)
- POST `/topup` -- Top up partner balance
- POST `/check-balance` -- Check partner balance

### AccountController
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/AccountController.java`
- **Base:** `/` (root-level paths)
- POST `/create-account` -- Create billing account
- POST `/recharge` -- Recharge account

### RateController
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/RateController.java`
- **Base:** `/` (root-level paths, uses `@ApiController`)
- POST `/rate-task` -- Get all rate tasks
- POST `/new-task` -- Add new rate task

### AuditLogController
- **File:** `FreeSwitchREST/src/main/java/freeswitch/controller/audit/AuditLogController.java`
- **Base:** `/api/audit`

---

## FreeSwitchEsl Service

**Port:** 5070 | **Eureka name:** `FreeSwitchEsl`

### ESL Connection
- `EslStarter` (CommandLineRunner) connects to FreeSWITCH at startup via `EslClient`
- `EslClient` uses `org.freeswitch.esl.client.inbound.Client` to connect to the ESL socket
- Connects to `eslIp:port` with password "ClueCon", subscribes to `plain all` events
- CCL config: ESL IP `103.95.96.98`, port `8021`, socket port `8084`

### ESL Event Handling
`EslEventListener` implements `IEslEventListener` and processes all `CHANNEL_*` events:

1. **CHANNEL_ANSWER** and **CHANNEL_HANGUP_COMPLETE** (events with >45 headers):
   - Serialized to JSON and **published to Kafka** on the ESL event topic
   - On CHANNEL_HANGUP_COMPLETE, stops the call state timer in `FreeswitchApi.callStateTrackers`

2. **CHANNEL_PARK**:
   - Triggers `CallAdmissionController.performCallAdmission()` -- runs the full call admission logic (partner auth, prefix matching, balance reservation, MNP lookup, digit manipulation, LCR routing, concurrent call limits)

3. **CHANNEL_BRIDGE**: Logged only
4. **CHANNEL_UNPARK**: Logged only

### Kafka Topics
- **ESL Event Topic** (configurable via `NewTopic` bean): Receives serialized CHANNEL_ANSWER and CHANNEL_HANGUP_COMPLETE events
- **`config_event_loader`** (consumer): Triggers config reload in ConfigManager when config changes are pushed
- **`messageTopic1`** (consumer): Receives charging events (ChargingStart/ChargingEnd) for prepaid balance reserve/return

### Key Components
- **CallAdmissionController** (`eslInbound/CallAdmissionController.java`): Core call routing logic -- partner authentication, prefix matching, balance checks, MNP, digit rules, LCR
- **FreeswitchApi** (`eslInbound/FreeswitchApi.java`): Direct ESL API wrapper for sending commands to FreeSWITCH
- **Rating package** (`Rating/`): Rate calculation, prefix matching, billing span handling
- **NumberRules** (`NumberRules/`): Caller/callee number manipulation rules (add prefix, cut digits, DID assignment)

---

## WebSocket Service

**Project:** `WebSocket-SpringBoot`

The WebSocket service implements **WebRTC-to-SIP bridging** for browser-based calling. Key components:

- **Call Signaling Stacks:**
  - `JingleStack` / `JingleChannel` -- Jingle (XMPP) based WebRTC signaling
  - `VertoStack` / `VertoChannel` -- FreeSWITCH Verto protocol for WebRTC-SIP bridging
  - `FreeswithcCaller` -- Direct FreeSWITCH origination

- **Call State Machine:** `CallStateMachine` / `CallStateMachineBuilder` -- Manages call lifecycle states (IDLE, RINGING, ANSWERED, HOLD, CONFERENCE, HANGUP)

- **Call Bridge:** `CallBridge` / `OriginatingBridge` -- Bridges between WebRTC and SIP legs

- **SDP Handling:** `SDPParser`, `SDPMessageFactory` -- SDP offer/answer negotiation

- **ICE:** `IceCandidateExtractor` -- ICE candidate gathering for NAT traversal

- **Protocols:** Supports Jingle (XMPP) and Verto (FreeSWITCH native WebRTC) signaling

**Note:** The main `Websocket.java` and `CallManager.java` are currently **commented out** -- the service uses the newer `calling/` package architecture.

---

## CCL Configuration

### FreeSwitchREST (`application-ccl76.properties`)

| Setting | Value |
|---------|-------|
| Calling prefix | `09646` |
| MySQL DB host | `103.95.96.99` |
| MySQL database | `telcobright` |
| Homer DB | `homer_data` |
| MySQL user | `tbuser` |
| ESL host | `127.0.0.1` (localhost, same machine) |
| ESL port | `8021` |
| ESL password | `ClueCon` |
| PostgreSQL (FusionPBX) | `103.95.96.100:5432/fusionpbx` |
| FusionPBX REST API | `https://103.95.96.100/app/rest_api/rest.php` |
| Kafka bootstrap | `localhost:9092` |
| Email (SMTP) | `email.cosmocom.net:465` (SSL) |
| Email from | `noreply.iptelephony@cosmocom.net` |
| Portal URL | `https://iptsp.cosmocom.net` |
| Invoice company | Cosmocom Limited |
| Recording path | `/var/recordings/` |
| SIP OPTIONS auto-discovery | `true` |

### FreeSwitchEsl (`application-ccl.properties`)

| Setting | Value |
|---------|-------|
| FS name | `ccl_FreeSwitch` |
| ESL IP | `103.95.96.98` |
| ESL port | `8021` |
| Socket port | `8084` |
| User XML path | `/usr/local/freeswitch/conf/directory/default` |
| MySQL DB host | `10.10.199.10` |
| MySQL database | `telcobright` |
| PostgreSQL (FusionPBX) | `114.130.145.82:5432/fusionpbx` |
| Kafka bootstrap | `localhost:9092` |

### CclFsConfig (Java hardcoded defaults)

| Setting | Value |
|---------|-------|
| ESL IP | `103.95.96.98` |
| Reserve balance | `false` |
| Modify caller number | `true` |
| Perform MNP lookup | `true` |

### API Gateway (`application-ccl76.properties`)

| Setting | Value |
|---------|-------|
| R2DBC MySQL | `103.95.96.77:3306/telcobright` |
| Gateway port | `8001` |

### API Gateway Routing
The gateway uses Eureka service discovery. FreeSwitchREST is registered as `FREESWITCHREST` in Eureka, so all requests to `/FREESWITCHREST/**` are routed to port 5071. Public (unauthenticated) endpoints include partner CRUD, dashboard endpoints, and some CDR endpoints.

---

## Database Entities (Voice)

### MySQL (`telcobright` database) -- Core voice entities:

| Entity | Table | Purpose |
|--------|-------|---------|
| `Partner` | `partner` | Voice partners / IPTSP clients |
| `PartnerPrefix` | `partnerprefix` | Partner IP/prefix auth mappings |
| `Route` | `route` | Voice routes (outbound route definitions) |
| `RouteMetaData` | `routemetadata` | Route metadata (gateway info, weight, priority) |
| `Dialplan` | `dialplan` | Dial plan definitions |
| `DialplanRoute` | `dialplanroute` | Dial plan to route mappings |
| `DialplanPrefix` | `dialplanprefix` | Dial plan prefix matching rules |
| `CallSrc` | `callsrc` | Call source / trunk definitions |
| `DidPool` | `didpool` | DID number pools |
| `DidNumber` | `didnumber` | Individual DID numbers |
| `DidAssignment` | `didassignment` | DID-to-partner assignments |
| `DidAssignmentHistory` | `didassignmenthistory` | DID assignment audit trail |
| `Cdr` | `cdr` | Call Detail Records (MySQL legacy) |
| `CdrState` | `cdr_state` | CDR processing state tracking |
| `Account` | `account` | Billing accounts |
| `AccountReserve` | `accountreserve` | Balance reservations during calls |
| `PackageAccount` | `packageaccount` | Prepaid package accounts |
| `PackagePurchase` | `packagepurchase` | Package purchase history |
| `CallRecording` | `callrecording` | Call recording file index |
| `ConcurentChannelUse` | `concurentchanneluse` | Concurrent channel usage snapshots |
| `DigitFilterPlan` | `digitfilterplan` | Digit manipulation filter plans |
| `DigitFilterRule` | `digitfilterrule` | Individual digit filter rules |
| `DigitCut` | `digitcut` | Digit cut/replace rules |
| `MnpEntity` | `mnp` | Mobile Number Portability lookups |
| `RateTask` | `ratetask` | Rating task definitions |
| `Rate` | `rate` | Call rate definitions |
| `AuditLog` | `audit_log` | System audit trail |
| `VoiceBroadcastingCampaign` | `voicebroadcastingcampaign` | Voice broadcast campaign logs |

### PostgreSQL (`fusionpbx` database) -- FusionPBX entities:

| Entity | Table | Purpose |
|--------|-------|---------|
| `VDomain` | `v_domains` | FusionPBX tenant domains |
| `VExtension` | `v_extensions` | SIP extensions (with call forward, DND, follow-me fields) |
| `VConference` | `v_conferences` | Conference rooms |
| `VXmlCdr` | `v_xml_cdr` | FusionPBX XML CDR records |

---

## Key Files for Customization

### Service Layer (FreeSwitchREST)
- `/FreeSwitchREST/src/main/java/freeswitch/service/FsControllerService.java` -- Core live call & concurrent call logic (ESL commands)
- `/FreeSwitchREST/src/main/java/freeswitch/service/FusionPbxActiveCallService.java` -- Active call ESL operations
- `/FreeSwitchREST/src/main/java/freeswitch/service/FusionPbxGatewayService.java` -- Gateway CRUD via FusionPBX PHP API
- `/FreeSwitchREST/src/main/java/freeswitch/service/FusionPbxRegistrationService.java` -- SIP registration queries
- `/FreeSwitchREST/src/main/java/freeswitch/service/FusionPbxCallCenterService.java` -- Call center ACD operations
- `/FreeSwitchREST/src/main/java/freeswitch/service/FusionPbxCallRecordingService.java` -- Call recording via PHP API
- `/FreeSwitchREST/src/main/java/freeswitch/service/CdrService.java` -- Legacy MySQL CDR queries
- `/FreeSwitchREST/src/main/java/freeswitch/service/QosService.java` -- QoS/CSSR/call-drop calculations
- `/FreeSwitchREST/src/main/java/freeswitch/service/SipOptionsMonitorService.java` -- SIP OPTIONS health checks

### ESL Layer (FreeSwitchEsl)
- `/FreeSwitchEsl/src/main/java/freeswitch/eslInbound/EslEventListener.java` -- ESL event handler & Kafka publisher
- `/FreeSwitchEsl/src/main/java/freeswitch/eslInbound/CallAdmissionController.java` -- Call admission & routing logic
- `/FreeSwitchEsl/src/main/java/freeswitch/eslInbound/FreeswitchApi.java` -- ESL command wrapper
- `/FreeSwitchEsl/src/main/java/freeswitch/Rating/RatingRule.java` -- Call rating engine
- `/FreeSwitchEsl/src/main/java/freeswitch/NumberRules/CallNumberModifyRule.java` -- Number manipulation rules
- `/FreeSwitchEsl/src/main/java/freeswitch/config/CclFsConfig.java` -- CCL-specific FreeSWITCH config

### Configuration
- `/FreeSwitchREST/src/main/resources/application-ccl76.properties` -- CCL REST API config
- `/FreeSwitchEsl/src/main/resources/application-ccl.properties` -- CCL ESL config
- `/API-Gateway/src/main/resources/application-ccl76.properties` -- CCL gateway R2DBC config
- `/API-Gateway/src/main/java/com/telcobright/gateway/security/publicendpoints/PublicEndpointRegistry.java` -- Public endpoint whitelist
