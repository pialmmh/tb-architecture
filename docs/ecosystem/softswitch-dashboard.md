# SOFTSWITCH_DASHBOARD -- Architecture Overview

## Summary

SOFTSWITCH_DASHBOARD is a multi-tenant React SPA that serves as the unified admin and client portal for Telcobright's softswitch platform. It provides dashboards, call management, SMS campaign management, voice broadcasting, CDR/billing reports, user/role management, and package management. The UI adapts per tenant profile (CCL, BTCL PBX, BTCL HCC) with tenant-specific branding, feature flags, API endpoints, and theme colors. It consumes three backend API gateways: FREESWITCHREST (voice/PBX operations), SMSREST (SMS operations via routesphere), and AUTHENTICATION (user/role management).

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Framework | React 18.3 (SPA) |
| Build System | Vite 6.x (`vite build` outputs to `build/`) |
| State Management | Redux (redux + react-redux + redux-thunk) |
| Routing | react-router-dom v6 (BrowserRouter) |
| UI Libraries | MUI v6, Bootstrap 5, Reactstrap, Ant Design 5, React-Feather, FontAwesome |
| Charts | ApexCharts, Recharts, ECharts, Chart.js |
| HTTP Client | Axios (with interceptors for JWT Bearer tokens) |
| WebSocket | Native WebSocket (not socket.io despite the dependency) |
| Auth | JWT (jwt-decode), Auth0 provider wrapper (likely legacy/unused -- actual login is email/password to custom backend) |
| Forms | React Hook Form + Formik + Yup validation |
| Styling | SCSS (Bootstrap extended), styled-components, MUI ThemeProvider, CSS variables for per-tenant theming |
| File Export | ExcelJS + file-saver |
| Deployment Runtime | PM2 + `serve` (static file server on port 3000) |
| Package Name | `desk_client` v2.0.0 |

## Directory Structure

```
SOFTSWITCH_DASHBOARD/
  package.json              # Build scripts with per-profile support
  vite.config.js            # Vite config, dev port 3000, allowed hosts for BTCL domains
  deploy.sh                 # Deploy to 103.95.96.98 (CCL production)
  deploy_76.sh              # Deploy to 103.95.96.76 (development server)
  deploy-pbx.sh             # Deploy to 114.130.145.82 (BTCL PBX)
  deploy-contact-center.sh  # Deploy to 114.130.145.75 LXC container (BTCL Contact Center)
  src/
    main.jsx                # Entry point: Auth0Provider > Redux Provider > ContactProvider > Layout > App
    App.jsx                 # MUI ThemeProvider + CCLContextProvider + WebSocketManager + CallHandler + Router
    Router.jsx              # All route definitions (60+ routes), JWT expiry check, lazy-loaded pages
    PrivateRoute.jsx        # ROLE_ADMIN guard (unused in current routing)
    configs/
      index.js              # Profile-based config loader (VITE_PROFILE env var)
      profiles/
        local.js            # localhost:8001
        ccl.js              # iptsp.cosmocom.net:8001
        btcl_pbx.js         # vbs.btcliptelephony.gov.bd:4000
        btcl_hcc.js         # hcc.btcliptelephony.gov.bd
      navigationConfig.jsx  # Legacy static nav config (not used by current sidebar)
    apiServices/
      apiClient.js          # Axios clients: freeswitchClient, smsClient, authClient
      [27 service directories]  # One per domain (Partner, CDR, SMS, Package, etc.)
    helpers/
      useApi.js             # Generic POST wrapper with Bearer token injection
    hooks/
      useActiveCalls.js     # Live call data hook (role-aware)
      usePartnerDetails.js  # Partner info fetcher
      useDistributors.js    # Distributor list
      usePagination.js      # Pagination state
      useSearchFilters.js   # Filter state
    services/
      WebSocketService.jsx  # Singleton WebSocket client for live calls (FREESWITCHREST/ws/live-calls)
    context/
      CClContext.jsx        # React Context for sidebar title and call stats filter
    extensions/
      access-control/       # AccessControl component
    redux/
      storeConfig/          # Redux store configuration
      actions/              # Action creators (auth, SMS, core, company, clients, etc.)
      reducers/             # Corresponding reducers
    layouts/
      components/
        menu/vertical-menu/sidemenu/SideMenuContent.jsx  # Role-based sidebar menu (hardcoded per role)
        navbar/Navbar.jsx   # Top navbar with role-aware elements
        footer/             # Footer component
    views/
      dashboard/
        analytics/AnalyticsDashboard.jsx  # Role-dispatched dashboard (BTRC, Admin, User, SMS, Voice, WebRTC)
        WebRtc/              # WebRTC calling: Calls, Contacts, CallScreen, CallHandler, WebSocketClient
      pages/
        authentication/login/BTRCLogin.jsx     # Login page shell
        authentication/login/BTRCLoginJWT.jsx  # Actual login form (POST to AUTHENTICATION/auth/login)
        misc/error/          # 404, 400, 500, Success, Cancel, Fail pages
      services/
        ActiveCalls/         # Live call monitoring
        PbxActiveCalls/      # FusionPBX active calls (all domains)
        ConcurrentCall/      # BTRC concurrent call view
        QoS/                 # Quality of Service reports
        Reports/CDRPbx/      # PBX CDR reports
        Reports/CDRBilling/  # Billing CDR reports
        SummaryReport/       # Domestic In/Out summary reports
        SMSFrontend/
          Admin/             # SMS admin: IncomingSms, SmsQueue, ContactGroup, DND, ForbiddenWords, SmsTemplate, SenderIdManagement, APIDocumentation
          Client/            # SMS client: SendSMS, SendDynamicSMS, Campaigns, SMSHistory, Reports, Balance, Policy
          SMSDashboard/      # SMS dashboard with live WebSocket feed
          SMSWebSocket/      # WebSocket client for SMS live data
        VoiceBroadcastFrontend/
          Client/            # Voice broadcast: SendVoiceBroadcast, Campaigns, History
          VoiceDashboard/    # Voice broadcast dashboard with live feed
        Partner&Routes/      # PartnerInfo, PartnerPrefix, Distributors, SmsRouting
        DidManagement/       # DID pools, assignments, history
        PackageManagement/   # Package CRUD, purchase, user packages, TopUp
        UserManagment/       # Users, SIP registrations
        RoleManagment/       # Role management
        RatePlanManagement/  # Rate plans, rates, rate plan assignments
        Routing/             # Routing plans, call sources
        DialplanManager/     # Dial plans, dial plan prefixes, digit filter plans
        ACL/                 # Access control lists
        MnpPool/             # MNP (Mobile Number Portability) pool
        RetailPartner/       # Retail partner management
        CallRecordings/      # Call recording playback
        UserCallHistory/     # Per-user CDR history and summary
        AuditLogs/           # System audit logs
        SendMail/            # Email sending
        SopDocumentation/    # Dynamic SOP documentation viewer (per-tenant HTML files)
        ProfilePage/         # User profile page
```

## Key Components

### Authentication (`BTRCLoginJWT.jsx`)
- Login form POSTs `{email, password}` to `{root}/AUTHENTICATION/auth/login`
- If input is all digits, POSTs to `{root}/AUTHENTICATION/auth/webrtc/login` (WebRTC user login)
- Response stored in `localStorage.userInfo` as JSON containing `{token, ...}`
- JWT token expiry checked on app load in `Router.jsx` -- if expired, `localStorage` is cleared and user sees login
- Token attached to all API calls via `Authorization: Bearer {token}` header

### Role-Based UI (SideMenuContent.jsx)
The sidebar menu is entirely role-driven and hardcoded per role. Roles discovered in codebase:

| Role | Menu Context |
|------|-------------|
| `ROLE_ADMIN` | Full admin menu: dashboard, live calls, partners, routes, DIDs, packages, CDR, users, roles, ACL, audit logs, reports, documentation |
| `ROLE_RESELLER` | Subset of admin: voice calls, SMS, partner management, DID, packages, CDR, reports. Combined voice+SMS when both feature flags are on |
| `ROLE_USER` | End-user view: SMS-only or voice-only menus depending on feature flags. SMS: incoming, send, campaigns, history, reports, packages, policy, contacts. Voice: dashboard, active calls, packages, CDR, reports, SIP registrations |
| `ROLE_BTRC` | Regulatory view: BTRC portal, concurrent calls, QoS |
| `ROLE_WEBRTC` | WebRTC softphone: calls, contacts |
| `ROLE_SMSADMIN` | SMS admin menu |
| `ROLE_VOICE_BRODCUST` | Voice broadcast dashboard |
| `ROLE_READONLY` | Read-only admin dashboard |

### API Client Layer (`apiClient.js`)
Three axios instances with shared interceptors:

| Client | Base URL | Purpose |
|--------|----------|---------|
| `freeswitchClient` | `{root}/FREESWITCHREST/` | Voice/PBX operations, partners, CDR, live calls, routing, packages, DIDs |
| `smsClient` | `{root}/SMSREST/` | SMS campaigns, tasks, contacts, DND, policies, SMS dashboard |
| `authClient` | `{root}/AUTHENTICATION/` | User CRUD, login, roles, permissions |

The legacy `useApi.js` helper is still used by many services -- it wraps `axios.post()` with manual token injection.

### WebSocket for Live Data (`WebSocketService.jsx`)
- Connects to `{root}/FREESWITCHREST/ws/live-calls?token={jwt}`
- Uses wss:// when page is HTTPS, ws:// otherwise
- Subscription-based: UI components subscribe to endpoints like `/admin-live-calls-summary`, `/concurrent-call-profile`, etc.
- Auto-reconnect with exponential backoff (max 5 attempts)
- Singleton instance shared across the app

### SMS WebSocket (`SMSWebSocket/WebSocketClient.jsx`)
- Separate WebSocket client for SMS live dashboard data
- Used by `SMSDashboard/LiveSms.jsx` for real-time SMS delivery stats

## External Connections

| Target | Protocol | Purpose |
|--------|----------|---------|
| `{root}/FREESWITCHREST/` | HTTPS REST (POST) | Voice/PBX API gateway -- partners, CDR, live calls, routing, DIDs, packages, call recordings, MNP, ACL, dial plans, SIP registrations, dashboard stats |
| `{root}/SMSREST/` | HTTPS REST (POST) | SMS API gateway -- campaigns, SMS tasks, contacts, DND, policies, rate plans, forbidden words, SMS queue, balance, templates, time bands, retry config |
| `{root}/AUTHENTICATION/` | HTTPS REST (POST) | Auth API gateway -- login, user CRUD, roles, permissions |
| `{root}/FREESWITCHREST/ws/live-calls` | WebSocket (wss/ws) | Real-time live call data (concurrent calls, per-partner stats) |
| SMS WebSocket (endpoint varies) | WebSocket | Real-time SMS delivery stats for dashboard |
| FusionPBX domains | Via FREESWITCHREST | PBX active calls across all domains |

### Key API Endpoints Consumed

**AUTHENTICATION gateway:**
- `auth/login` -- user login
- `auth/webrtc/login` -- WebRTC user login
- `auth/createUser`, `editUser`, `deleteUser`, `getUsers`, `getUser`, `getUserByEmail`
- Role/permission management endpoints

**FREESWITCHREST gateway:**
- `partner/get-partners`, `partner/get-partner`, `partner/create-partner`, `partner/update-partner`, `partner/delete-partner`
- `admin/DashBoard/getIntervalWiseCall`, `getTotalCall`, `getOutgoingCall`, `getIncomingCall`, `getMissedCall`
- `user/DashBoard/getIntervalWiseCall`, `getTotalCall`, etc.
- `getConcurrentCall`, `concurrent-call-profile`, `concurrent-call-partner-wise`
- `admin-live-calls-summary`, `user-live-calls-summary`
- `analytics/graph-data`, `analytics/graph-data/csv`
- `qos-reports/call-drop`, `qos-reports/cssr`
- `mcc-reports/max-call-periodically`
- `admin/DashBoard/top5PartnerCallCounts`, `admin/DashBoard/system-info`
- `api/sofia/registrations`, `api/sofia/partner-wise-registrations`
- `api/ip-auth/create`, `api/ip-auth/update`, `api/ip-auth/delete`, `api/ip-auth/get-ip-by-idauthuser`
- `get-retail-partners`
- `route/*` -- SMS routing
- `sip-options-monitor/status`

**SMSREST gateway:**
- `SmsTask/sendSms`
- `campaign/save-campaign`, `campaign/get-campaigns`, `campaign/enableCampaign`, `campaign/disableCampaign`
- `campaignTask/*`
- `contact-group/*`, `contact-item/*`
- `dnd-group/*`, `dnd-list/*`
- `api/policies/*`
- `api/timeBands`
- `api/retry-cause-codes`, `api/retry-intervals`
- `api/smserrors/allsmserrors`
- `api/sms-queue/`
- `api/admin/dashboard/*`
- `enum-job-status/`

## Data Flow

1. **User Authentication**: User submits email/password on login page. POST to `AUTHENTICATION/auth/login`. JWT token returned, stored in `localStorage.userInfo`. Token decoded to determine role, which drives sidebar menu and dashboard selection.

2. **Dashboard Rendering**: `AnalyticsDashboard.jsx` checks user role and feature flags (`voice`, `sms`) to render the appropriate dashboard (Admin, User, BTRC, SMS, Voice Broadcast, or WebRTC).

3. **Live Call Data**: `WebSocketService` singleton connects to FREESWITCHREST WebSocket. Components subscribe to specific endpoints. Server pushes updates that are dispatched to registered callbacks.

4. **SMS Flow**: SMS client pages (SendSMS, SendDynamicSMS) POST to `SMSREST/SmsTask/sendSms`. Campaign management through `SMSREST/campaign/*`. Live SMS stats via dedicated WebSocket.

5. **CDR/Reports**: Reports pages POST date ranges and filters to FREESWITCHREST endpoints. Response data rendered in MUI DataGrid tables with Excel export capability (ExcelJS).

6. **Partner Management**: Admin creates/manages partners via FREESWITCHREST `/partner/*` endpoints. Partners are referenced by `idPartner` throughout the system.

## Configuration

### Profile System (`src/configs/index.js`)

The app uses build-time profile selection via `VITE_PROFILE` environment variable:

| Profile | `VITE_PROFILE` | API Root | Features |
|---------|----------------|----------|----------|
| Local | `local` | `http://localhost:8001` | voice=1, sms=0 |
| CCL | `ccl` (default) | `https://iptsp.cosmocom.net:8001` | voice=1, sms=0 |
| BTCL PBX | `btcl_pbx` | `https://vbs.btcliptelephony.gov.bd:4000` | voice=1, sms=0 |
| BTCL HCC | `btcl_hcc` | `https://hcc.btcliptelephony.gov.bd` | voice=1, sms=0 |

Each profile defines: API root URLs (`root`, `root2`), branding (logos, favicons), feature flags (`voice`, `sms`), titles, documentation paths, and full theme color palette (primary, sidebar, header colors).

### Feature Flags
- `voice`: Enables voice/PBX features (active calls, CDR, SIP registrations, routing)
- `sms`: Enables SMS features (send SMS, campaigns, SMS history, SMS dashboard)
- `login`: Controls which logo/branding is shown on login page (1=voice branding, 0=SMS branding)

### Theme System
Theme colors from profile are injected as CSS variables on `document.documentElement` at app startup and as MUI theme via `ThemeProvider`. This enables per-tenant visual branding without code changes.

### Documentation
Each profile can specify tenant-specific SOP documentation HTML files served from the `public/` directory (e.g., `/CCL_IPTSP_Admin_SOP.html`, `/BTCL_PBX_Admin_SOP.html`).

## Deployment

### Build
```bash
# Default (CCL profile):
npm run build

# Specific profile:
VITE_PROFILE=btcl_pbx npm run build
# or
npm run build:ccl
npm run build:btcl
```

Output directory: `build/`

### Deploy Scripts

| Script | Target Server | Profile | Remote Path | Notes |
|--------|--------------|---------|-------------|-------|
| `deploy.sh` | 103.95.96.98 (CCL production) | default | `/home/telcobright/Documents/Production/DASHBOARD_CCL` | PM2 app: `softswitch-dashboard` |
| `deploy_76.sh` | 103.95.96.76 (development) | default (config patched via sed) | `/home/telcobright/Documents/Development/SOFTSWITCH_DASHBOARD` | Backs up/restores config.json during build |
| `deploy-pbx.sh` | 114.130.145.82 (BTCL PBX) | `btcl_pbx` | `/home/telcobright/SOFTSWITCH_DASHBOARD` | Verifies via `https://hippbx.btcliptelephony.gov.bd:3001/` |
| `deploy-contact-center.sh` | 114.130.145.75 (host) -> ContactCenter LXC (10.10.192.79) | default | `/home/SOFTSWITCH_DASHBOARD` inside container | Pushes via `lxc file push`, PM2 app: `contact-center-admin-dashboard` |

All deployments:
1. Build locally (`npm run build` or `VITE_PROFILE=X npm run build`)
2. rsync `build/` to remote server
3. Restart PM2 process running `serve -s build -p 3000`

Served as static files by `serve` on **port 3000** in all environments.

### Verification
Deploy scripts verify HTTP 200 from the running app after restart.

## Integration Points with Other Projects

### routesphere-core (Quarkus backend)
- The SMSREST gateway endpoints (`/SMSREST/*`) are served by routesphere-core's REST API
- SMS campaigns, tasks, contacts, policies, rate plans, DND, forbidden words, SMS queue -- all managed by routesphere
- The dashboard's SMS WebSocket connects to routesphere for live delivery stats
- `campaignTask/*` endpoints map to routesphere's campaign task management

### FREESWITCHREST (Java backend / API gateway)
- Voice/PBX operations: partner management, CDR, live calls, routing, DIDs, packages, ACL, dial plans
- WebSocket at `/ws/live-calls` for real-time concurrent call data
- SIP registration monitoring via FreeSWITCH Sofia module
- Call recordings served through this gateway
- QoS reports (call drop, CSSR) and concurrent call analytics

### AUTHENTICATION (Java backend)
- User authentication (JWT issuance)
- User CRUD operations
- Role and permission management
- Partner-user association (`getUserByEmail` returns `idPartner`)

### API Gateway Architecture
All three backend services (FREESWITCHREST, SMSREST, AUTHENTICATION) are accessed through a single host:port defined by the profile's `root` URL. This implies an API gateway or reverse proxy (e.g., nginx at port 8001 for CCL, port 4000 for BTCL PBX) that routes by URL path prefix:
- `/FREESWITCHREST/*` -> FreeSWITCH REST backend
- `/SMSREST/*` -> routesphere-core
- `/AUTHENTICATION/*` -> Auth backend

### config-manager
- Not directly consumed by the dashboard, but config-manager's CDC events trigger changes in routesphere that affect SMS delivery behavior visible in the dashboard

### sigtran
- Not directly consumed by the dashboard. Sigtran handles the actual SS7/MAP signaling for SMS delivery. The dashboard shows delivery results that flow back through routesphere.
