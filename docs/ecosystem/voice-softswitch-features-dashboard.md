# Voice/Softswitch Features -- SOFTSWITCH_DASHBOARD

## CCL Profile Configuration

**File:** `src/configs/profiles/ccl.js`

| Setting | Value |
|---------|-------|
| `voice` | `1` (enabled) |
| `sms` | `0` (disabled) |
| `login` | `1` (enabled) |
| API root | `https://iptsp.cosmocom.net:8001` |
| Branding | `CCL_Logo.png`, Navy/Steel Blue theme (`#2B5A8A`) |
| Company | Cosmopolitan Communication Limited |
| Documentation | Admin SOP: `/CCL_IPTSP_Admin_SOP.html`, User SOP: `/CCL_IPTSP_User_Partner_SOP.html` |

**Profile loading:** `src/configs/index.js` reads `VITE_PROFILE` env var (defaults to `ccl`). The `voice` and `sms` flags control which sidebar menus, dashboard components, and routes are rendered.

---

## Voice Pages & Routes

### 1. Main Dashboard (`/`)
- **Component:** `src/views/dashboard/analytics/AnalyticsDashboard.jsx`
- **What it does:** Role-based dashboard dispatcher. For `voice=1`:
  - `ROLE_ADMIN` (default/else) and `ROLE_READONLY` -> `SuperAdmin/Dashboard.jsx`
  - `ROLE_USER` with `voice=1, sms=0` -> `SuperAdmin/Dashboard.jsx`
  - `ROLE_RESELLER` with `voice=1` -> `SuperAdmin/Dashboard.jsx`
  - `ROLE_WEBRTC` -> `WebRtcHomePage.jsx`
  - `ROLE_VOICE_BRODCUST` -> `VoiceBroadcastFrontend/VoiceDashboard/VoiceDashboard.jsx`
- **Dashboard sub-components** (all in `src/views/services/Dashboard/Pages/SuperAdmin/`):
  - `LiveCalls.jsx` -- Live ICX/gateway call counts (AGNI.DOM, AGNI.INT, BDIX, BTCL, SCOM, LOCAL)
  - `PartnerLiveCalls.jsx` -- Per-partner concurrent calls table (WebSocket real-time)
  - `PartnerLiveCallsSummary.jsx` -- Summary of live calls per partner
  - `DashboardLiveCallsTable.jsx` -- Active calls table embedded in dashboard
  - `CallStatus.jsx` -- Total/outgoing/incoming/missed call stat cards
  - `CallSummaryChart.jsx` -- Interval-wise call summary bar chart (Recharts)
  - `ConcurrentCallGraph.jsx` -- Concurrent call analytics graph with CSV export
  - `StatCard.jsx` -- Reusable stat card (total, active, outbound, inbound, missed)
  - `ICXStatus.jsx` -- Gateway SIP OPTIONS monitor (health status of ICX gateways, auto-refresh 30s)
  - `PBXOverview.jsx` -- FreeSWITCH server CPU/memory usage, top processes
  - `TopPartner.jsx` -- Top 5 partners by call count
  - `DashboardBalance.jsx` -- Balance display
  - `DashboardPackageBalance.jsx` -- Package balance display
  - `DashboardCdrTable.jsx` -- Recent CDR table on dashboard
- **API endpoints called (via WebSocket):**
  - `/admin-live-calls-summary` (ROLE_ADMIN)
  - `/user-live-calls-summary` (ROLE_USER, with idPartner)
  - `/concurrent-call-partner-wise` (ROLE_ADMIN, ROLE_READONLY, ROLE_RESELLER)
  - `/concurrent-call-profile` (ROLE_ADMIN)
  - `/concurrent-call-profile-partner` (ROLE_USER, with idPartner)
- **API endpoints called (REST):**
  - `admin/DashBoard/getIntervalWiseCall`, `admin/DashBoard/getTotalCall`, `admin/DashBoard/getOutgoingCall`, `admin/DashBoard/getIncomingCall`, `admin/DashBoard/getMissedCall`
  - `admin/DashBoard/system-info` (PBX overview)
  - `admin/DashBoard/top5PartnerCallCounts`
  - `analytics/graph-data` (concurrent call graph)
  - `sip-options-monitor/status` (ICX gateway health)
  - `getConcurrentCall` (live calls count)

### 2. Live/Active Calls (`/liveCalls`)
- **Component:** `src/views/services/ActiveCalls/ActiveCalls.jsx` -> `ActiveCallsTable.jsx`
- **What it does:** Real-time table of all active calls across the switch, using WebSocket subscription via `useActiveCalls` hook
- **Hook:** `src/hooks/useActiveCalls.js` -- subscribes to WebSocket endpoints based on role
- **API:** WebSocket endpoints `/concurrent-call-profile` (admin) or `/concurrent-call-profile-partner` (user)

### 3. PBX Active Calls (`/pbxActiveCalls`)
- **Component:** `src/views/services/PbxActiveCalls/PbxActiveCalls.jsx` -> `PbxActiveCallsTable.jsx`
- **What it does:** FusionPBX active calls across all domains. Auto-refreshes every 5 seconds via REST polling. Supports call hangup action.
- **API endpoints:**
  - `api/v1/active-calls/list` -- list all active calls
  - `api/v1/active-calls/details` -- get call details by UUID
  - `api/v1/active-calls/hangup` -- hangup a call by UUID
- **Note:** Hidden from sidebar for `btcl_hcc` profile

### 4. CDR PBX (`/CDRPbx`)
- **Component:** `src/views/services/Reports/CDRPbx/CDRPbx.jsx`
- **What it does:** FusionPBX-sourced call detail records. Stats cards (total, inbound, outbound, internal, answered, missed, avg duration). Filters: date range (24h/7d/30d/custom), direction, status, hangup cause, caller/destination. Export to XLSX.
- **API endpoints:**
  - `admin/getCallHistory` (ROLE_ADMIN)
  - `user/getCallHistory` (ROLE_USER)
  - `get-partner-prefix-by-email` (user prefix lookup)
  - `admin/downloadCallHistory` / `user/downloadCallHistory` (export)

### 5. CDR Billing (`/CDRBilling`)
- **Component:** `src/views/services/Reports/CDRBilling/CDRBilling.jsx`
- **What it does:** Billing CDR from the softswitch billing engine. Advanced filters: partner, ingress/egress partner, routes, ANS ID, service group, caller/called numbers, date range. Export to XLSX.
- **API endpoints:**
  - `api/cdr/cdrByFilterForAdmin` / `api/cdr/cdrByFilterForUser`
  - `api/cdr/cdrByFilterExportForAdmin` / `api/cdr/cdrByFilterExportForUser`

### 6. Call Recordings (`/callRecordings`)
- **Component:** `src/views/services/CallRecordings/CallRecordings.jsx` + `AudioPlayerBar.jsx`
- **What it does:** Browse, search, play, and download call recordings. Filters: caller, destination, date range. In-browser audio playback with play/stop controls. Export recordings as ZIP.
- **API endpoints:**
  - `api/recordings/get-recordings` -- paginated recording list
  - `api/recordings/stream` -- stream audio for playback
  - `api/recordings/auto-load` -- auto-sync recordings from FS
  - `api/recordings/download-zip` -- export filtered recordings as ZIP

### 7. Concurrent Call Reports (`/concurrentCall`)
- **Component:** `src/views/services/ConcurrentCall/ConcurrentCall.jsx`
- **What it does:** Wraps the `MaxConcurrentChart` component from BTRC dashboard. Shows max concurrent call trends.
- **API:** `mcc-reports/max-call-periodically`

### 8. QoS Reports (`/qos`)
- **Component:** `src/views/services/QoS/Qos.jsx`
- **What it does:** Quality of Service reports with combined bar chart showing Call Drop Rate and Call Success Ratio (CSSR). Filters: date range picker, period selection (hourly/daily/monthly). Uses Recharts.
- **API endpoints:**
  - `qos-reports/call-drop`
  - `qos-reports/cssr`

### 9. SIP Registration Status (`/usersSipRegistration`)
- **Component:** `src/views/services/UserManagment/UsersSipRegistration/UsersSipRegistration.jsx`
- **What it does:** Shows all SIP accounts with registration status (online/offline). Stats: total, registered, offline. Search and filter by status. Expandable rows for SIP account details (user agent, IP, etc.).
- **API:** User service endpoints for SIP registration data

### 10. SIP Accounts / Retail Partners (`/retailPartner`)
- **Component:** `src/views/services/RetailPartner/RetailPartner.jsx`
- **What it does:** Manage SIP trunk accounts (retail partners). CRUD operations for SIP endpoints.

### 11. Dial Plan Management (`/dialplan`)
- **Component:** `src/views/services/DialplanManager/DialPlan/DialPlan.jsx` + `DialplanModal.jsx`
- **What it does:** CRUD for dial plans. Fields: name, description, priority, call source, sequential/load-balanced, inbound DID flag.
- **API endpoints:**
  - `create-dialplan`, `update-dialplan`, `get-dialplans`, `get-dialplan-by-id`, `delete-dialplan`

### 12. Dial Plan Prefix (`/dialplanPrefix`)
- **Component:** `src/views/services/DialplanManager/DialPlanPrefix/DialPlanPrefix.jsx` + `DialPlanPrefixModal.jsx`
- **What it does:** Map called/calling prefixes to dial plans with percentage-based distribution. Links to call sources and partners.
- **API endpoints:**
  - `create-dialplan-prefix`, `update-dialplan-prefix`, `get-dialplan-prefixes`, `delete-dialplan-prefix`, `dialplan-prefix/exists`

### 13. Dial Plan Route (nested within DialPlan)
- **Component:** `src/views/services/DialplanManager/DialPlanRoute/DialPlanRoute.jsx` + `DialPlanRouteModal.jsx` + `DialPlanRouteTable.jsx`
- **What it does:** Assign routes to dial plans with priority and optional digit filter plan.
- **API endpoints:**
  - `add-route-to-dialplan`, `delete-dialplan-route`

### 14. Digit Filter Plan (`/digitFilterPlan`)
- **Component:** `src/views/services/DialplanManager/DigitFilterPlan/DigitFilterPlan.jsx`
- **Sub-components:** `DigitCut/`, `DigitFilterPlanDetail/`
- **What it does:** Manage digit manipulation plans (prefix stripping, digit cutting) applied to dial plan routes.
- **API endpoints:**
  - `api/plans/list`, `api/plans/create`, `api/plans/update`, `api/plans/delete`, `api/plans/getById`

### 15. DID Management (`/didPool`, `/didAssignments`, `/didAssignmentHistory`, `/didPoolNumber`, `/AssignDid/:poolId`)
- **Components:**
  - `src/views/services/DidManagement/DidPool/DidPool.jsx` -- DID pool management (create/edit/delete pools)
  - `src/views/services/DidManagement/DidPool/DidAssign.jsx` -- Assign DIDs within a pool
  - `src/views/services/DidManagement/DidAssignments/DidAssignments.jsx` -- View/manage DID-to-partner assignments
  - `src/views/services/DidManagement/DidAssingmentHistory/DisAssingmentHistory.jsx` -- DID assignment audit history
  - `src/views/services/DidManagement/DidPoolNumber/DidPoolNumber.jsx` -- Individual DID number management
- **API endpoints:**
  - `get-did-pools`, `create-did-pool`, `update-did-pool`, `delete-did-pool`
  - `get-did-assignments`, `create-did-assignment`, `update-did-assignment`, `delete-did-assignment`
  - `get-did-numbers`, `get-did-numbers-by-pool`, `create-did-number`, `update-did-number`
  - `get-did-assignment-by-didpool`, `get-did-assignment-by-partner-id`
  - `create-did-assign-from-csv` (CSV import)
  - `get-did-assignment-history`

### 16. Call Source (`/callSource`)
- **Component:** `src/views/services/Routing/CallSource/CallSource.jsx`
- **What it does:** Manage call sources (ingress gateways/trunks). CRUD operations.
- **API endpoints:** `create-callsrc`, `update-callsrc`, `get-callsrcs`

### 17. Routing Plan (`/routingPlan`)
- **Component:** `src/views/services/Routing/RoutingPlan/RoutingPlan.jsx`
- **What it does:** Manage voice routing plans.

### 18. Partner Prefix (`/partnerPrefixes`)
- **Component:** `src/views/services/Partner&Routes/PartnerPrefix/PartnerPrefixes.jsx`
- **What it does:** Manage partner-specific number prefixes for routing/CDR matching.

### 19. MNP Pool (`/mnpPool`)
- **Component:** `src/views/services/MnpPool/MnpPool.jsx`
- **What it does:** Mobile Number Portability lookup pool. View MNP data, file counts, export.
- **API endpoints:** `api/mnp_pull`, `api/mnp_pull/file-count`, `api/mnp_pull/export-file-list`

### 20. Summary Reports (`/domesticIn`, `/domesticOut`)
- **Components:**
  - `src/views/services/SummaryReport/DomesticIn/DomesticIn.jsx`
  - `src/views/services/SummaryReport/DomesticOut/DomesticOut.jsx`
- **What it does:** IPTSP domestic inbound/outbound summary reports.
- **API endpoints:** `api/summry-reports/dom-in-iptsp`, `api/summry-reports/dom-out-iptsp`

### 21. User Call History (`/userCallHistory`, `/userCDRSummery/:didNumber`)
- **Components:**
  - `src/views/services/UserCallHistory/UserCallHistory.jsx`
  - `src/views/services/UserCallHistory/UserCDRSummery.jsx`
- **What it does:** End-user call history and per-DID CDR summary.

### 22. WebRTC Softphone (`/call-page-webrtc`, `/contatc-page-webrtc`, `/call-screen`)
- **Components (all in `src/views/dashboard/WebRtc/`):**
  - `Calls.jsx` -- Main calls page with dialpad and call history
  - `CallScreen.jsx` -- Full-screen call UI (rendered full-layout, no sidebar)
  - `Dialpad.jsx` -- DTMF dial pad (0-9, *, #) with call/hangup buttons
  - `CallHandler.jsx` -- SIP/WebRTC call state machine
  - `CallDialog.jsx` -- Call dialog overlay
  - `CallPopup.jsx` -- Popup during active call
  - `CallsHistory.jsx` -- Call history list
  - `CallState.js` -- Call state enum/constants
  - `Contacts.jsx`, `ContactsUser.jsx`, `ContactsCategory.jsx`, `ContactModal.jsx`, `ContactContext.jsx` -- Contact management
  - `IncomingCallModal.jsx` -- Incoming call notification modal
  - `ToasterIncoming.jsx`, `ToasterOngoing.jsx`, `ToasterOngoingForDialPad.jsx` -- Toast notifications for calls
  - `WebSocketClient.jsx` -- WebRTC-specific WebSocket client
  - `WebSocketManager.js` -- WebRTC WebSocket connection manager
  - `WebRtcHomePage.jsx` -- Landing page for ROLE_WEBRTC users
  - `Responses.js` -- Response handling constants
- **API endpoints (WebRTC contacts):**
  - `contact/get-contacts`, `contact/create-contact`, `contact/delete-contact`, `contact/update-contact`
  - `get-call-history`

### 23. Voice Broadcasting (`/sendVoiceBroadcast`, `/voiceBroadcastCampaigns`, `/voiceBroadcastHistory`)
- **Components (in `src/views/services/VoiceBroadcastFrontend/`):**
  - `Client/VoicBroadcastTask/SendVoiceBroadcast/SendVoiceBroadcast.jsx` -- Create and send voice broadcast
  - `Client/VoicBroadcastTask/VoiceBroadcastCampaigns/VoiceBroadcastCampaigns.jsx` -- Campaign list
  - `Client/VoicBroadcastTask/VoiceBroadcastHistory/VoiceBroadcastHistory.jsx` -- Broadcast history
  - `VoiceDashboard/VoiceDashboard.jsx` -- Dashboard for ROLE_VOICE_BRODCUST
  - `Client/Policy/` -- Voice broadcast retry policies (cause codes, retry interval, time band)

### 24. CCL-Specific Data Services
- **Files:**
  - `src/apiServices/CCLDataServices/getCclIpTraffic.js` -- CCL IP traffic: `get-partner-details`, `get-outgoing-calls`
  - `src/apiServices/CCLDataServices/getCclTdmTraffic.js` -- CCL TDM (BTCL) traffic: `get-btcl-calls`

### 25. ACL (`/ACL`)
- **Component:** `src/views/services/ACL/ACL.jsx`
- **What it does:** Access Control Lists for SIP gateway/trunk access rules.

---

## Voice API Services

All voice API services communicate with the backend via `freeswitchClient` (base URL: `{root}/FREESWITCHREST/`).

| Service File | Purpose | Key Endpoints |
|---|---|---|
| `CallsServices/CallsServices.js` | Live calls, concurrent call profiles, ICX status | `concurrent-call-profile`, `admin-live-calls-summary`, `sip-options-monitor/status` + WebSocket configs |
| `CDRServices/CDRServices.js` | Call detail records (PBX + billing) | `admin/getCallHistory`, `user/getCallHistory`, `api/cdr/cdrByFilterForAdmin`, exports |
| `CallRecordingsServices/CallRecordingsServices.js` | Call recording browse/play/export | `api/recordings/get-recordings`, `api/recordings/stream`, `api/recordings/download-zip` |
| `DialPlanServices/DialPlanServices.js` | Dial plan CRUD + prefix + routes | `create-dialplan`, `get-dialplans`, `create-dialplan-prefix`, `add-route-to-dialplan` |
| `DIDPoolServices/DidPoolServices.js` | DID pool/number/assignment CRUD | `get-did-pools`, `create-did-pool`, `create-did-assignment`, `create-did-assign-from-csv` |
| `DigitFilterPlanServices/digitFilterPlanervices.js` | Digit filter plan CRUD | `api/plans/list`, `api/plans/create`, `api/plans/update` |
| `DigitFilterPlanServices/DigitCut.js` | Digit cut operations | Nested within digit filter plans |
| `DigitFilterPlanServices/DigitFilterPlanDetails.js` | Digit filter plan detail operations | Plan detail CRUD |
| `CallSourceServices/CallSourceServices.js` | Call source (ingress gateway) CRUD | `create-callsrc`, `update-callsrc`, `get-callsrcs` |
| `PbxActiveCallsServices/PbxActiveCallsServices.js` | FusionPBX active call list/hangup | `api/v1/active-calls/list`, `api/v1/active-calls/hangup` |
| `WebRtcServices/getWebRtcServices.js` | WebRTC contacts & call history | `contact/get-contacts`, `contact/create-contact`, `get-call-history` |
| `AdminDashboardServices/adminDashboardServices.js` | Dashboard stats, QoS, concurrent graphs | `admin/DashBoard/*`, `analytics/graph-data`, `qos-reports/*`, `mcc-reports/*` |
| `UserDashboardServices/userDashboardServices.js` | User dashboard stats, balance | `getTotalCallForUser`, `user/DashBoard/getBalance` |
| `SummaryReportServices/SummaryReportServices.js` | IPTSP summary reports | `api/summry-reports/dom-out-iptsp`, `api/summry-reports/dom-in-iptsp` |
| `MnpServices/MnpServices.js` | MNP pool lookup | `api/mnp_pull`, `api/mnp_pull/file-count` |
| `CCLDataServices/getCclIpTraffic.js` | CCL IP traffic data | `get-partner-details`, `get-outgoing-calls` |
| `CCLDataServices/getCclTdmTraffic.js` | CCL TDM/BTCL traffic | `get-btcl-calls` |

**API Client architecture** (`src/apiServices/apiClient.js`):
- Three gateway-specific axios clients: `freeswitchClient` (`/FREESWITCHREST/`), `smsClient` (`/SMSREST/`), `authClient` (`/AUTHENTICATION/`)
- All voice features use `freeswitchClient`
- Automatic JWT token injection via request interceptor

---

## Voice Navigation (per role)

Defined in `src/layouts/components/menu/vertical-menu/sidemenu/SideMenuContent.jsx`. The menu is built by `getInitialState(authRole)` which checks `config.voice` and `config.sms` flags.

### ROLE_ADMIN / default (`voice=1`, else block)
1. Dashboard (`/`)
2. Active Calls (`/liveCalls`)
3. PBX Active Calls (`/pbxActiveCalls`) -- hidden for `btcl_hcc`
4. Partner (`/distributors`)
5. Route (`/smsRouting`)
6. Call Source (`/callSource`)
7. Manage DID -> DID Pool, DID Assignments, Assignment History, DID Pool Number
8. Dial Plan -> Dial Plan, Dial Plan Prefix, Digit Filter Plan
9. Manage Package -> Package, Package Purchase, User Packages, TopUp
10. Manage Rateplan -> Rate Plan, Rate Plan Assignment
11. CDR -> CDR Pbx, CDR Billing
12. Report -> Domestic In, Domestic Out
13. SIP Accounts (`/retailPartner`)
14. SIP Registrations (`/usersSipRegistration`)
15. Call Recordings (`/callRecordings`)
16. Users (`/users`)
17. MNP Pool (`/mnpPool`)
18. Audit Logs (`/auditLogs`)
19. Documentation (`/documentation`)
20. Send Mail (`/sendMail`)

### ROLE_USER (`voice=1, sms=0`)
1. Dashboard (`/`)
2. Active Calls (`/liveCalls`)
3. All Packages (`/allPackages`)
4. Purchase History (`/purchaseHistory`)
5. CDR (`/CDRBilling`)
6. Report -> Domestic In, Domestic Out
7. SIP Registrations (`/usersSipRegistration`)
8. Documentation (`/userdocumentation`)

### ROLE_RESELLER (`voice=1, sms=0`)
1. Dashboard (`/`)
2. Active Calls (`/liveCalls`)
3. Partner (`/distributors`)
4. Route (`/smsRouting`)
5. Call Source (`/callSource`)
6. Manage DID -> DID Pool, DID Assignments, Assignment History, DID Pool Number
7. Manage Package -> Package, Package Purchase, User Packages, TopUp
8. Manage Rateplan -> Rate Plan, Rate Plan Assignment
9. CDR -> CDR Billing
10. Report -> Domestic In, Domestic Out
11. SIP Accounts (`/retailPartner`)
12. SIP Registrations (`/usersSipRegistration`)
13. Users (`/users`)

### ROLE_READONLY
1. Dashboard (`/`)
2. Active Calls (`/liveCalls`)
3. Manage Package -> Package Purchase, User Packages
4. SIP Accounts (`/retailPartner`)
5. Call Recordings (`/callRecordings`)
6. Audit Logs (`/auditLogs`)

### ROLE_BTRC
1. BTRC Portal (`/`)
2. Concurrent Call (`/concurrentCall`)
3. QoS (`/qos`)

### ROLE_WEBRTC
1. Calls (`/call-page-webrtc`)
2. Contacts (`/contatc-page-webrtc`)

### ROLE_VOICE_BRODCUST
- Voice Broadcast Dashboard (`/`)
- (Additional VBS menu items loaded contextually)

---

## WebSocket / Live Calls

### WebSocket Service (`src/services/WebSocketService.jsx`)
- **Singleton** pattern -- single WebSocket connection shared app-wide
- **URL:** `wss://{root}/FREESWITCHREST/ws/live-calls?token={jwt}`
- **Features:**
  - Auto-reconnect with exponential backoff (max 5 attempts, 3s base delay)
  - Subscription-based model: subscribe to endpoints with optional `idPartner` filter
  - Resubscribes all subscriptions on reconnect
  - Connection state tracking: CONNECTING, CONNECTED, CLOSING, DISCONNECTED

### WebSocket Hook (`src/helpers/useWebSocketLiveData.js`)
- React hook wrapping WebSocketService
- Accepts array of subscription configs: `[{ endpoint, idPartner }]`
- Returns: `{ results[], loading, errors[], connectionState }`
- Auto-connects with JWT token, handles loading state until first data arrives

### Active Calls Hook (`src/hooks/useActiveCalls.js`)
- Role-aware hook for live call data
- ROLE_ADMIN: subscribes to `/concurrent-call-profile`
- ROLE_USER: subscribes to `/concurrent-call-profile-partner` with idPartner
- Dashboard view limits USER data to 3 rows

### WebSocket Endpoints Used
| Endpoint | Description | Used By |
|----------|-------------|---------|
| `/admin-live-calls-summary` | Overall live call summary (admin) | Dashboard |
| `/user-live-calls-summary` | Per-partner live call summary | Dashboard (USER) |
| `/concurrent-call-partner-wise` | Partner-wise concurrent calls | Dashboard (ADMIN/READONLY) |
| `/concurrent-call-profile` | Full concurrent call profile | Active Calls (ADMIN) |
| `/concurrent-call-profile-partner` | Partner-specific concurrent profile | Active Calls (USER) |

---

## Voice Dashboard Components

Located in `src/views/services/Dashboard/Pages/SuperAdmin/`:

| Component | Description |
|-----------|-------------|
| `Dashboard.jsx` | Main orchestrator -- assembles all dashboard widgets, WebSocket subscriptions |
| `StatCard.jsx` | Animated stat cards: Total Calls, Active, Outbound, Inbound, Missed |
| `LiveCalls.jsx` | ICX gateway live stats: AGNI.DOM, AGNI.INT, BDIX, BTCL, SCOM, LOCAL (or CATALEYA for btcl profiles) |
| `CallStatus.jsx` | Total/outgoing/incoming/missed call counts for current period |
| `CallSummaryChart.jsx` | Bar chart of interval-wise call distribution |
| `ConcurrentCallGraph.jsx` | Time-series concurrent call analytics with CSV export |
| `PartnerLiveCalls.jsx` | Real-time per-partner concurrent call table (WebSocket) |
| `PartnerLiveCallsSummary.jsx` | Aggregated partner live call summary |
| `DashboardLiveCallsTable.jsx` | Active calls table embedded in dashboard view |
| `ICXStatus.jsx` | Gateway SIP OPTIONS health check (auto-refresh 30s) |
| `PBXOverview.jsx` | FreeSWITCH server CPU, memory, top processes |
| `TopPartner.jsx` | Top 5 partners by call count |
| `DashboardBalance.jsx` | Account balance display (prepaid/postpaid) |
| `DashboardPackageBalance.jsx` | Package-specific balance |
| `DashboardCdrTable.jsx` | Recent CDR table |
| `AccountDetails.jsx` | Partner account info |

Located in `src/views/services/Dashboard/Pages/BtrcAdmin/`:
| `MaxConcurrentChart.jsx` | Max concurrent call chart (used in `/concurrentCall` route) |
| `BtrcPortal.jsx` | BTRC regulatory portal dashboard |

---

## Key Files for Customization

### Config & Branding
- `src/configs/profiles/ccl.js` -- CCL-specific config (API root, logos, theme colors, feature flags)
- `src/configs/index.js` -- Profile loader and config export

### Routing
- `src/Router.jsx` -- All route definitions (add/remove voice pages here)

### Navigation / Sidebar
- `src/layouts/components/menu/vertical-menu/sidemenu/SideMenuContent.jsx` -- Role-based sidebar menus (3371 lines, all roles hardcoded)

### API Layer
- `src/apiServices/apiClient.js` -- Axios client with `freeswitchClient` base URL
- `src/apiServices/CallsServices/CallsServices.js` -- Live calls + WebSocket subscription configs
- `src/apiServices/CDRServices/CDRServices.js` -- CDR queries and exports
- `src/apiServices/CallRecordingsServices/CallRecordingsServices.js` -- Recording playback and export
- `src/apiServices/DialPlanServices/DialPlanServices.js` -- Dial plan CRUD
- `src/apiServices/DIDPoolServices/DidPoolServices.js` -- DID management
- `src/apiServices/AdminDashboardServices/adminDashboardServices.js` -- Dashboard data + QoS

### Real-Time / WebSocket
- `src/services/WebSocketService.jsx` -- Singleton WebSocket client
- `src/helpers/useWebSocketLiveData.js` -- React hook for WebSocket subscriptions
- `src/hooks/useActiveCalls.js` -- Role-aware live call hook

### Dashboard Components
- `src/views/services/Dashboard/Pages/SuperAdmin/` -- All voice dashboard widgets
- `src/views/dashboard/analytics/AnalyticsDashboard.jsx` -- Dashboard dispatcher

### Voice Feature Pages
- `src/views/services/ActiveCalls/` -- Live active calls
- `src/views/services/PbxActiveCalls/` -- FusionPBX active calls with hangup
- `src/views/services/Reports/CDRPbx/` -- PBX CDR
- `src/views/services/Reports/CDRBilling/` -- Billing CDR
- `src/views/services/CallRecordings/` -- Recording playback/export
- `src/views/services/DialplanManager/` -- Dial plan, prefix, route, digit filter
- `src/views/services/DidManagement/` -- DID pools, numbers, assignments
- `src/views/services/Routing/` -- Call source, routing plan
- `src/views/services/QoS/` -- QoS reports
- `src/views/services/ConcurrentCall/` -- Concurrent call reports
- `src/views/services/SummaryReport/` -- Domestic in/out reports
- `src/views/services/UserCallHistory/` -- User call history
- `src/views/services/RetailPartner/` -- SIP accounts
- `src/views/services/UserManagment/UsersSipRegistration/` -- SIP registration status

### WebRTC Softphone
- `src/views/dashboard/WebRtc/` -- Full WebRTC softphone (dialpad, call screen, contacts, history)

### Voice Broadcasting
- `src/views/services/VoiceBroadcastFrontend/` -- Voice broadcast campaigns, history, dashboard

### Deploy
- `deploy.sh` -- Deploys to CCL server at `103.95.96.98`, path `/home/telcobright/Documents/Production/DASHBOARD_CCL`, served via PM2 + `serve` on port 3000
