# L3venture UI Routes & Navigation

> Source: `SOFTSWITCH_DASHBOARD` React app
> Deployed at: **sms.l3ventures.net**
> Generated: 2026-03-20

## Config

| Setting | Value |
|---------|-------|
| Deploy profile (`vite_profile`) | `local` (from `tools/deploy/tenant-conf-v2/l3venture.conf`) |
| Profile config file | `src/configs/profiles/local.js` |
| `voice` (feature flag) | `0` (disabled) -- effective on l3venture server |
| `sms` (feature flag) | `1` (enabled) -- effective on l3venture server |
| `login` | `1` (voice-style login page branding, but SMS mode content) |
| API root | `http://localhost:8001` (reverse-proxied behind sms.l3ventures.net) |
| Serve port | `3000` |

**Note:** The `local` profile in the repo has `voice: 1, sms: 0`, but the l3venture production deployment effectively runs with `voice: 0, sms: 1`. This may be achieved by modifying the built config at deploy time or via a server-specific profile override. The navigation and dashboard rendering below reflects the **effective** l3venture config (`voice=0, sms=1`).

## Profile System

The config loader (`src/configs/index.js`) selects a profile based on `VITE_PROFILE` env var at build time:

| Profile | Tenant | voice | sms | login |
|---------|--------|-------|-----|-------|
| `local` | L3venture (deploy conf) | 1* | 0* | 1 |
| `ccl` | Cosmopolitan Communication | 1 | 0 | 1 |
| `btcl_pbx` | BTCL PBX | 1 | 0 | 1 |
| `btcl_hcc` | BTCL HCC | 1 | 0 | 1 |

*L3venture overrides to voice=0, sms=1 at deploy/runtime.

---

## Route Map

All routes defined in `src/Router.jsx`. Every route requires authentication (JWT in localStorage). Unauthenticated users see only `/` (login) and `/accessDenied`.

### Core Routes

| Path | Component | Feature Area | Description |
|------|-----------|-------------|-------------|
| `/` | `AnalyticsDashboard` | Dashboard | Role-aware: shows SMSDashboard for ROLE_USER (sms=1), ROLE_SMSADMIN, ROLE_RESELLER (sms=1) |
| `/smsDashboard` | `SmsDashboard` | SMS Dashboard | Dedicated SMS dashboard page |
| `/profilePage` | `ProfilePage` | User Profile | User profile management |

### SMS Portal -- Admin

| Path | Component | Feature Area | Description |
|------|-----------|-------------|-------------|
| `/incomingSMS` | `IncomingSMS` | SMS Admin | View incoming SMS messages |
| `/smsQueue` | `SmsQueue` | SMS Admin | SMS queue management and monitoring |
| `/contactGroup` | `ContactGroup` | SMS Admin | Manage contact groups for campaigns |
| `/dndGroup` | `DNDGroup` | SMS Admin | Do-Not-Disturb group management |
| `/smsTemplate` | `SmsTemplate` | SMS Admin | SMS template management |
| `/forbiddenWords` | `ForbiddenWords` | SMS Admin | Manage forbidden/blocked words |

### SMS Portal -- Client

| Path | Component | Feature Area | Description |
|------|-----------|-------------|-------------|
| `/sendSms` | `SendSMS` | SMS Task | Send ad-hoc SMS messages |
| `/sendDynamicSMS` | `SendDynamicSMS` | SMS Task | Send dynamic/personalized SMS via CSV |
| `/campaigns` | `Campaigns` | SMS Task | View/manage SMS campaigns |
| `/campaignOverview/:campaignId` | `CampaignsOverview` | SMS Task | Campaign detail view with task progress |
| `/smsHistory` | `SmsHistory` | SMS History | SMS delivery history and logs |
| `/reports` | `Reports` | SMS Reports | SMS delivery reports and analytics |
| `/apiDocumentation` | `ApiDocumentation` | SMS Docs | API documentation for SMS integration |
| `/price` | `Price` (Balance) | SMS Billing | View balance/pricing info |
| `/policy` | `Policy` | SMS Policy | View SMS policies and terms |

### Partner & Route Management

| Path | Component | Feature Area | Description |
|------|-----------|-------------|-------------|
| `/distributors` | `Distributors` | Partners | Partner (distributor) CRUD |
| `/partnerInfo/:idPartner` | `PartnerInfo` | Partners | Partner detail view |
| `/smsRouting` | `SmsRouting` | Routing | SMS routing rules management |
| `/partnerPrefixes` | `PartnerPrefixes` | Routing | Partner prefix management |
| `/callSource` | `CallSource` | Routing | Call source definitions |
| `/routingPlan` | `RoutingPlan` | Routing | Routing plan management |

### Sender ID / DID Management

| Path | Component | Feature Area | Description |
|------|-----------|-------------|-------------|
| `/didPool` | `DidPool` | Sender ID | SenderID pool management |
| `/AssignDid/:poolId` | `AssignDid` | Sender ID | Assign DIDs from a pool |
| `/didPoolNumber` | `DidPoolNumbers` | Sender ID | DID/SenderID number management |
| `/didAssignments` | `DidAssignments` | Sender ID | SenderID assignment to partners |
| `/didAssignmentHistory` | `DidAssignmentsHistory` | Sender ID | Assignment audit trail |
| `/senderIdPool` | `SenderIdPool` | Sender ID | (Alternate) SenderID pool |
| `/senderIdNumber` | `SenderIdNumbers` | Sender ID | (Alternate) SenderID numbers |
| `/senderIdAssignments` | `SenderIdAssignments` | Sender ID | (Alternate) SenderID assignments |
| `/senderIdAssignmentHistory` | `SenderIdAssignmentsHistory` | Sender ID | (Alternate) Assignment history |

### Package Management

| Path | Component | Feature Area | Description |
|------|-----------|-------------|-------------|
| `/package` | `Package` | Packages | Package definition CRUD |
| `/packageItem/:packageId` | `PackageItemAssign` | Packages | Assign items to packages |
| `/allPackages` | `AllPackages` | Packages | Browse all available packages |
| `/userPackages` | `UserPackages` | Packages | User's purchased packages |
| `/packagePurchase` | `PackagePurchase` | Packages | Purchase a package |
| `/purchaseHistory` | `PurchaseHistory` | Packages | Purchase history |
| `/topUp` | `TopUp` | Packages | Account top-up |

### Rate Plan Management

| Path | Component | Feature Area | Description |
|------|-----------|-------------|-------------|
| `/ratePlan` | `RatePlan` | Rate Plans | Rate plan CRUD |
| `/rates/:idRatePlan` | `Rates` | Rate Plans | View/edit rates within a plan |
| `/ratePlanAssignment` | `RatePlanAssignment` | Rate Plans | Assign rate plans to partners |

### CDR & Reports

| Path | Component | Feature Area | Description |
|------|-----------|-------------|-------------|
| `/CDRBilling` | `CDRBilling` | CDR | Billing CDR records |
| `/CDRPbx` | `CDRPbx` | CDR | PBX CDR records |
| `/domesticIn` | `DomesticIn` | Reports | Domestic inbound summary report |
| `/domesticOut` | `DomesticOut` | Reports | Domestic outbound summary report |

### Voice Features (hidden on l3venture -- voice=0)

| Path | Component | Feature Area | Description |
|------|-----------|-------------|-------------|
| `/liveCalls` | `LiveCalls` | Voice | Active/live calls monitor |
| `/pbxActiveCalls` | `PbxActiveCalls` | Voice | FusionPBX active calls |
| `/userCallHistory` | `UserCallHistory` | Voice | User call history |
| `/userCDRSummery/:didNumber` | `UserCDRSummery` | Voice | CDR summary per DID |
| `/callRecordings` | `CallRecordings` | Voice | Call recordings browser |
| `/retailPartner` | `RetailPartner` | Voice | SIP account management |
| `/usersSipRegistration` | `UsersSipRegistration` | Voice | SIP registration status |
| `/concurrentCall` | `ConcurrentCall` | BTRC | Concurrent call monitoring |
| `/qos` | `Qos` | BTRC | Quality of Service metrics |

### Voice Broadcast (separate role)

| Path | Component | Feature Area | Description |
|------|-----------|-------------|-------------|
| `/sendVoiceBroadcast` | `SendVoiceBroadcast` | Voice BC | Create voice broadcast campaign |
| `/voiceBroadcastCampaigns` | `VoiceBroadcastCampaigns` | Voice BC | Voice broadcast campaign list |
| `/voiceBroadcastHistory` | `VoiceBroadcastHistory` | Voice BC | Voice broadcast history |

### Dial Plan Management

| Path | Component | Feature Area | Description |
|------|-----------|-------------|-------------|
| `/dialplan` | `Dialplan` | Dial Plan | Dial plan CRUD |
| `/dialplanPrefix` | `DialplanPrefix` | Dial Plan | Dial plan prefix management |
| `/digitFilterPlan` | `DigitFilterPlan` | Dial Plan | Digit filter plan management |

### User & System Admin

| Path | Component | Feature Area | Description |
|------|-----------|-------------|-------------|
| `/users` | `Users` | Admin | User management (CRUD, role assignment) |
| `/addRole` | `AddRole` | Admin | Role management |
| `/ACL` | `ACL` | Admin | Access control lists |
| `/mnpPool` | `MnpPool` | Admin | MNP (Mobile Number Portability) pool |
| `/auditLogs` | `AuditLogs` | Admin | System audit logs |
| `/sendMail` | `SendMail` | Admin | Send email to users |

### Documentation

| Path | Component | Feature Area | Description |
|------|-----------|-------------|-------------|
| `/documentation` | `SopDocumentation` | Docs | Admin SOP documentation (profile-specific) |
| `/userdocumentation` | `SopDocumentation` | Docs | User/partner SOP documentation |

### WebRTC (not used on l3venture)

| Path | Component | Feature Area |
|------|-----------|-------------|
| `/call-screen` | `CallScreen` | WebRTC |
| `/call-page-webrtc` | `CallWebrtc` | WebRTC |
| `/contatc-page-webrtc` | `ContactsWebrtc` | WebRTC |

### Payment & Error Pages

| Path | Component | Description |
|------|-----------|-------------|
| `/payment/success` | `Success` | Payment success callback |
| `/payment/cancel` | `Cancel` | Payment cancelled callback |
| `/payment/fail` | `Fail` | Payment failed callback |
| `/500` | `InternalServerError` | Server error page |
| `/400` | `BadRequest` | Bad request page |
| `/notAuthorized` | `NotAuthorized` | Access denied page |
| `/accessDenied` | `AccessDenied` | Login access denied |
| `*` | `NotFound` | 404 page |

---

## Navigation (Sidebar Menu)

The sidebar menu is defined entirely in `src/layouts/components/menu/vertical-menu/sidemenu/SideMenuContent.jsx` via the `getInitialState()` function. Menu structure depends on two inputs:

1. **User role** (`authRoles[0].name` from JWT)
2. **Feature flags** (`config.voice` and `config.sms`)

For l3venture (voice=0, sms=1), the relevant role+mode combinations are:

---

### ROLE_ADMIN (else branch -- no voice/sms check)

The ROLE_ADMIN menu is the **default/else** branch and is NOT affected by voice/sms flags. It shows the full admin menu:

1. Dashboard `/`
2. Active Calls `/liveCalls`
3. PBX Active Calls `/pbxActiveCalls` (hidden for btcl_hcc profile)
4. Partner `/distributors`
5. Route `/smsRouting`
6. Call Source `/callSource`
7. Manage DID (collapse)
   - Did Pool `/didPool`
   - Did Assignments `/didAssignments`
   - Assignment History `/didAssignmentHistory`
   - DidPool Number `/didPoolNumber`
8. Dial Plan (collapse)
   - Dial Plan `/dialplan`
   - Dial Plan Prefix `/dialplanPrefix`
   - Digit Filter Plan `/digitFilterPlan`
9. Manage Package (collapse)
   - Package `/package`
   - Package Purchase `/packagePurchase`
   - User Packages `/userPackages`
   - TopUp `/topUp`
10. Manage Rateplan (collapse)
    - Rate Plan `/ratePlan`
    - Rate Plan Assignment `/ratePlanAssignment`
11. CDR (collapse)
    - CDR Pbx `/CDRPbx`
    - CDR Billing `/CDRBilling`
12. Report (collapse)
    - Domestic In `/domesticIn`
    - Domestic Out `/domesticOut`
13. SIP Accounts `/retailPartner`
14. SIP Registrations `/usersSipRegistration`
15. Call Recordings `/callRecordings`
16. Users `/users`
17. MNP Pool `/mnpPool`
18. Audit Logs `/auditLogs`
19. Documentation `/documentation`
20. Send Mail `/sendMail`

---

### ROLE_SMSADMIN

Dedicated SMS admin menu (not affected by voice/sms flags):

1. Dashboard `/`
2. Partner `/distributors`
3. Route `/smsRouting`
4. Manage Sender Id (collapse)
   - Did Pool `/didPool`
   - Sender Id Assignments `/didAssignments`
   - Assignment History `/didAssignmentHistory`
5. Dial Plan (collapse)
   - Dial Plan `/dialplan`
   - Dial Plan Prefix `/dialplanPrefix`
   - Digit Filter Plan `/digitFilterPlan`
6. Manage Package (collapse)
   - Package `/package`
   - Package Purchase `/packagePurchase`
   - User Packages `/userPackages`
   - TopUp `/topUp`
7. Manage Rateplan (collapse)
   - Rate Plan `/ratePlan`
   - Rate Plan Assignment `/ratePlanAssignment`
8. CDR (collapse)
   - CDR Pbx `/CDRPbx`
   - CDR Billing `/CDRBilling`
9. Incoming SMS `/incomingSMS`
10. Sms Queue `/smsQueue`
11. SMS Task (collapse)
    - Send Adhoc SMS `/sendSms`
    - Campaigns `/campaigns`
    - SMS History `/smsHistory`
    - Reports `/reports`
    - Sms Template `/smsTemplate`
    - Api Documentation `/apiDocumentation`
12. Policy `/policy`
13. Forbidden Words `/forbiddenWords`
14. Contact Group `/contactGroup`
15. DND Group `/dndGroup`
16. Users `/users`

---

### ROLE_RESELLER (voice=0, sms=1)

When sms=1 and voice=0, ROLE_RESELLER gets an SMS-focused menu:

1. Dashboard `/`
2. SMS Task (collapse)
   - SMS History `/smsHistory`
   - Reports `/reports`
   - Api Documentation `/apiDocumentation`
3. Partners `/distributors`
4. Manage Sender Id (collapse)
   - SenderID Pool `/didPool`
   - Sender Id Assignments `/didAssignments`
   - Assignment History `/didAssignmentHistory`
   - SenderId Numbers `/didPoolNumber`
5. Manage Package (collapse)
   - Package `/package`
   - Package Purchase `/packagePurchase`
   - User Packages `/userPackages`
   - TopUp `/topUp`
6. Manage Rateplan (collapse)
   - Rate Plan `/ratePlan`
   - Rate Plan Assignment `/ratePlanAssignment`
7. CDR (collapse)
   - CDR Billing `/CDRBilling`
8. Forbidden Words `/forbiddenWords`
9. Contact Group `/contactGroup`
10. Users `/users`

---

### ROLE_USER (sms=1, voice=0)

SMS client user menu:

1. Dashboard `/`
2. Incoming SMS `/incomingSMS`
3. SMS Task (collapse)
   - Send Adhoc SMS `/sendSms`
   - Send Dynamic SMS `/sendDynamicSMS`
   - Campaigns `/campaigns`
   - SMS History `/smsHistory`
   - Reports `/reports`
   - Api Documentation `/apiDocumentation`
4. Manage Package (collapse)
   - Package Purchase `/packagePurchase`
5. Price `/Price`
6. Policy `/policy`
7. Contact Group `/contactGroup`

---

### ROLE_VOICE_BRODCUST

Voice broadcast user menu (not l3venture-relevant, but exists):

1. Dashboard `/`
2. Voice Broadcast Task (collapse)
   - Send Broadcast `/sendVoiceBroadcast`
   - Broadcast Campaigns `/voiceBroadcastCampaigns`
   - Broadcast History `/voiceBroadcastHistory`
   - Reports `/reports`
   - Api Documentation `/apiDocumentation`
3. Manage Sender (collapse)
   - Did Pool `/didPool`
   - Did Assignments `/didAssignments`
   - Assignment History `/didAssignmentHistory`
   - DidPool Number `/didPoolNumber`
4. Manage Package (collapse)
   - Package `/package`
   - Package Purchase `/packagePurchase`
   - User Packages `/userPackages`
   - TopUp `/topUp`
5. CDR (collapse)
   - CDR Pbx `/CDRPbx`
   - CDR Billing `/CDRBilling`
6. Dial Plan (collapse)
   - Dial Plan `/dialplan`
   - Dial Plan Prefix `/dialplanPrefix`
   - Digit Filter Plan `/digitFilterPlan`
7. Manage Rateplan (collapse)
   - Rate Plan `/ratePlan`
   - Rate Plan Assignment `/ratePlanAssignment`
8. Policy `/policy`
9. Contact Group `/contactGroup`

---

### ROLE_BTRC

BTRC regulator portal (3 items, no voice/sms flag dependency):

1. BTRC Portal `/`
2. Concurrent Call `/concurrentCall`
3. Qos `/qos`

---

### ROLE_WEBRTC

WebRTC consultant (no voice/sms flag dependency):

1. Username (display only)
2. Calls `/call-page-webrtc`
3. Contacts `/contatc-page-webrtc`

---

### ROLE_READONLY

Read-only user (no voice/sms flag dependency):

1. Dashboard `/`
2. Active Calls `/liveCalls`
3. Manage Package (collapse)
   - Package Purchase `/packagePurchase`
   - User Packages `/userPackages`
4. SIP Accounts `/retailPartner`
5. Call Recordings `/callRecordings`
6. Audit Logs `/auditLogs`

---

## Voice vs SMS Mode

The `voice` and `sms` flags from the profile config control **two things**:

### 1. Sidebar Menu Content

In `SideMenuContent.jsx`, the `getInitialState()` function checks `config.voice` and `config.sms` for roles: **ROLE_USER** and **ROLE_RESELLER** only. Other roles (ROLE_ADMIN, ROLE_SMSADMIN, ROLE_BTRC, ROLE_WEBRTC, ROLE_READONLY, ROLE_VOICE_BRODCUST) have **fixed menus** regardless of voice/sms flags.

| Role | voice=1, sms=0 | voice=0, sms=1 | voice=1, sms=1 |
|------|---------------|---------------|---------------|
| ROLE_USER | Voice dashboard: Active calls, packages, CDR, reports, SIP reg, docs | SMS dashboard: Incoming SMS, SMS Task (send/campaigns/history/reports/API docs), packages, price, policy, contacts | N/A (no combined branch) |
| ROLE_RESELLER | Full voice admin: dashboard, active calls, partner, route, call source, DID mgmt, packages, rateplans, CDR, reports, SIP, users | SMS-focused: SMS task, partners, sender ID mgmt, packages, rateplans, CDR, forbidden words, contacts, users | Both Voice Call (collapse) + SMS (collapse) with full menu for each |

### 2. Dashboard Component Selection

In `AnalyticsDashboard.jsx`, the `/` route renders different dashboards:

| Role | Condition | Dashboard Component |
|------|-----------|-------------------|
| ROLE_ADMIN | Always | `Dashboard` (SuperAdmin voice dashboard) |
| ROLE_BTRC | Always | `BtrcPortal` |
| ROLE_RESELLER + voice=1 | voice flag | `Dashboard` |
| ROLE_RESELLER + sms=1 | sms flag | `SMSDashboard` |
| ROLE_USER + sms=1, voice=0 | After BTRC agreement | `SMSDashboard` |
| ROLE_USER + voice=1, sms=0, login=1 | | `Dashboard` |
| ROLE_READONLY | Always | `Dashboard` |
| ROLE_SMSADMIN | Always | `SMSDashboard` |
| ROLE_VOICE_BRODCUST | Always | `VoiceDashboard` |
| ROLE_WEBRTC | Always | `WebRtcHomePage` |

### 3. Branding (login flag)

The `login` flag controls branding rather than access:
- `login=1`: Uses voice-style logo (`siteLogoUrl`), voice favicon, voice title
- `login!=1`: Uses SMS-style logo (`smsSiteLogoUrl`), SMS favicon, SMS title
- `login=2`: Special "aggregator" mode (renames "Reseller" to "Aggregator" in UI)

---

## Key Pages

### Dashboard (`/`)
- **SMS mode**: Shows `SMSDashboard` -- SMS delivery stats, live SMS counters, SMS history table, SMS summary charts
- **Voice mode**: Shows `Dashboard` -- concurrent call graphs, call status, CDR table, account balance details
- **API services**: `smsApiServices/SMSDashboard`, `AdminDashboardServices`, `UserDashboardServices`
- **ROLE_USER + sms=1**: BTRC compliance dialog shown on first visit (must agree to continue)

### SMS Dashboard (`/smsDashboard`)
- Dedicated SMS dashboard with live stats, history, and summary
- API: `smsApiServices/SMSDashboard`

### Send SMS (`/sendSms`)
- Ad-hoc SMS sending form: recipient number(s), sender ID, message content
- API: `smsApiServices/SmsTaskServices`

### Send Dynamic SMS (`/sendDynamicSMS`)
- CSV-based bulk SMS with personalized fields
- API: `smsApiServices/SmsTaskServices`

### Campaigns (`/campaigns`)
- SMS campaign list with status, progress, actions
- Click through to `/campaignOverview/:campaignId` for detail
- API: `smsApiServices/CampaignsServices`

### SMS History (`/smsHistory`)
- Delivery history with filters (date, status, sender ID, recipient)
- API: `smsApiServices/SmsTaskServices`

### Reports (`/reports`)
- SMS delivery reports with date range, aggregation
- API: `smsApiServices/SmsTaskServices`

### Partner Management (`/distributors`)
- CRUD for partner entities (SMS clients/resellers)
- Detail view at `/partnerInfo/:idPartner`
- API: `PartnerServices`

### SMS Routing (`/smsRouting`)
- Route rules: match patterns to SMS gateways
- API: `SmsRouteService`

### SenderID/DID Management (`/didPool`, `/didAssignments`, etc.)
- Pool management, number assignment to partners, history tracking
- In SMS mode, "DID" is used interchangeably with "SenderID"
- API: `DIDPoolServices`

### Package Management (`/package`, `/packagePurchase`, etc.)
- SMS credit packages: create, assign, purchase
- API: `PackageServices`

### Rate Plan Management (`/ratePlan`, `/ratePlanAssignment`)
- SMS rate plans and partner assignments
- API: `RateTaskServices`

### CDR Billing (`/CDRBilling`)
- Billing CDR records with search/filter
- API: `CDRServices`

### Users (`/users`)
- User CRUD with role assignment
- Roles available depend on config (sms=1 + login=2 shows "ROLE_AGGREGATOR")
- API: `UserServices`

### Contact Group (`/contactGroup`)
- Manage contact groups for SMS campaigns
- API: `smsApiServices/ContactGroupServices`

### DND Group (`/dndGroup`)
- Do-Not-Disturb lists
- API: `smsApiServices/DND`

### Policy (`/policy`)
- View SMS policies and terms of service
- API: `smsApiServices/Policy`

### Forbidden Words (`/forbiddenWords`)
- Manage blocked words list for SMS content filtering
- API: `smsApiServices/ForbiddenServices`

### SMS Queue (`/smsQueue`)
- Monitor SMS queue status and pending messages
- API: `smsApiServices/SmsQueue`

### SMS Template (`/smsTemplate`)
- Manage reusable SMS templates
- API: `smsApiServices/SmsTaskServices`

### API Documentation (`/apiDocumentation`)
- Interactive API docs for SMS HTTP API integration

### Audit Logs (`/auditLogs`)
- System audit trail
- API: `AuditLogsServices`

---

## Layout Architecture

### Layout Components
- **VerticalLayout** (default): Left sidebar + top navbar + content area
- **HorizontalLayout**: Top navigation bar + content area (not commonly used)
- **FullLayout**: No sidebar/navbar (used for login, error pages, call screen)

### Sidebar (`Sidebar.jsx`)
- Collapsible vertical menu
- Logo in header (switches between voice/SMS logo based on `config.login`)
- Menu items from `SideMenuContent.jsx` (role + feature flag driven)
- Perfect scrollbar for overflow

### Navbar (`Navbar.jsx`)
- Shows current page title from sidebar selection
- Partner name display for ROLE_USER and ROLE_RESELLER
- Customer type indicator (Prepaid/Postpaid)
- User dropdown with logout

### Content Area
- Wrapped in `Suspense` with spinner fallback
- Each route component is lazy-loaded for code splitting

---

## API Services Directory

All API services are in `src/apiServices/`:

| Service | Used By |
|---------|---------|
| `smsApiServices/SMSDashboard` | SMS Dashboard stats |
| `smsApiServices/SmsTaskServices` | Send SMS, campaigns, history, reports, templates |
| `smsApiServices/CampaignsServices` | Campaign CRUD |
| `smsApiServices/ContactGroupServices` | Contact group management |
| `smsApiServices/DND` | DND group management |
| `smsApiServices/ForbiddenServices` | Forbidden words |
| `smsApiServices/Policy` | Policy display |
| `smsApiServices/BalanceServices` | Balance/pricing |
| `smsApiServices/SmsQueue` | SMS queue monitoring |
| `smsApiServices/RatePlanServices` | SMS rate plans |
| `smsApiServices/SmsError` | SMS error codes |
| `smsApiServices/statusApiServices` | SMS status codes |
| `smsApiServices/TimeBands` | Time band config |
| `PartnerServices` | Partner CRUD |
| `SmsRouteService` | SMS routing |
| `DIDPoolServices` | DID/SenderID pool |
| `PackageServices` | Package management |
| `RateTaskServices` | Rate plan management |
| `CDRServices` | CDR records |
| `UserServices` | User management |
| `RoleServices` | Role management |
| `PermissionServices` | Permission management |
| `AuditLogsServices` | Audit logs |
| `SummaryReportServices` | Domestic in/out reports |
| `AdminDashboardServices` | Admin dashboard data |
| `UserDashboardServices` | User dashboard data |
| `CallsServices` | Active calls |
| `CallSourceServices` | Call source config |
| `CallRecordingsServices` | Call recordings |
| `DialPlanServices` | Dial plan management |
| `DigitFilterPlanServices` | Digit filter plans |
| `PartnerPrefixServices` | Partner prefixes |
| `MnpServices` | MNP pool |
| `RetailPartner` | SIP/retail partner |
| `PbxActiveCallsServices` | PBX active calls |
| `EmailServices` | Email sending |
| `WebRtcServices` | WebRTC calls |
| `CCLDataServices` | CCL-specific data |

All API calls use `src/apiServices/apiClient.js` which wraps axios with the configured `config.root` base URL and JWT token auth header.

---

## Feature Flag Summary for L3venture (sms.l3ventures.net)

| Feature | Available | Notes |
|---------|-----------|-------|
| SMS sending (adhoc/dynamic/campaign) | Yes | Core feature |
| SMS history & reports | Yes | All roles |
| SMS dashboard | Yes | ROLE_USER, ROLE_SMSADMIN, ROLE_RESELLER |
| Contact groups & DND | Yes | ROLE_SMSADMIN, ROLE_USER |
| SenderID management | Yes | Via DID management routes |
| Package management | Yes | All admin roles |
| Partner management | Yes | ROLE_ADMIN, ROLE_SMSADMIN, ROLE_RESELLER |
| Rate plan management | Yes | ROLE_ADMIN, ROLE_SMSADMIN, ROLE_RESELLER |
| Voice active calls | No | voice=0, hidden from menu |
| SIP registration | No | voice=0, hidden from menu |
| Call recordings | No | voice=0, hidden from menu |
| WebRTC | No | Separate role, not l3venture |
| Voice broadcast | No | Separate role (ROLE_VOICE_BRODCUST) |
| BTRC portal | No | Separate role (ROLE_BTRC) |
