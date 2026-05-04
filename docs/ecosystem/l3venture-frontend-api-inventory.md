# L3venture Frontend API Inventory

Complete inventory of every API endpoint called by the SOFTSWITCH_DASHBOARD frontend.

**Scanned codebase**: `/home/mustafa/telcobright-projects/SOFTSWITCH_DASHBOARD/src/`

**Date**: 2026-03-20

---

## API Client Configuration

The dashboard uses two API calling patterns:

1. **Gateway-specific axios instances** (`src/apiServices/apiClient.js`):
   - `freeswitchClient` — baseURL: `{root}/FREESWITCHREST/`
   - `smsClient` — baseURL: `{root}/SMSREST/`
   - `authClient` — baseURL: `{root}/AUTHENTICATION/`
   - All instances auto-attach `Authorization: Bearer {token}` from `localStorage.userInfo`
   - Response interceptor unwraps `response.data` automatically

2. **Legacy `apiCall.api()` helper** (`src/helpers/useApi.js`):
   - Uses raw `axios.post()` with full URL constructed in each service file
   - Also provides `apiCall.exportApi()` for blob/file downloads
   - Token passed explicitly as a parameter

**Base URL configuration** (`src/configs/index.js`):
- `root` and `root2` come from active profile (local: `http://localhost:8001`, ccl: `https://iptsp.cosmocom.net:8001`)
- For l3venture production, nginx proxies `https://sms.l3ventures.net` to `localhost:8001`

**Note on HTTP methods**: Nearly all API calls use POST, even reads/deletes. This is by design of the backend.

---

## AUTHENTICATION Gateway (`/AUTHENTICATION/`)

### Login / Authentication
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/AUTHENTICATION/auth/login` | `views/pages/authentication/login/BTRCLoginJWT.jsx` (inline) | Standard email/password login |
| POST | `/AUTHENTICATION/auth/webrtc/login` | `views/pages/authentication/login/BTRCLoginJWT.jsx` (inline) | WebRTC login (when email is numeric/extension) |

### User Management
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/AUTHENTICATION/getUsers` | `apiServices/UserServices/UserServices.js` | Fetch all users |
| POST | `/AUTHENTICATION/getUser` | `apiServices/UserServices/UserServices.js` | Fetch user by ID |
| POST | `/AUTHENTICATION/auth/createUser` | `apiServices/UserServices/UserServices.js` | Create new user |
| POST | `/AUTHENTICATION/editUser` | `apiServices/UserServices/UserServices.js` | Update user |
| POST | `/AUTHENTICATION/deleteUser` | `apiServices/UserServices/UserServices.js` | Delete user |
| POST | `/AUTHENTICATION/getUserByEmail` | `apiServices/AdminDashboardServices/adminDashboardServices.js` | Fetch partner details by email (user dashboard) |

### Role Management
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/AUTHENTICATION/auth/getRoles` | `apiServices/RoleServices/RoleServices.js` | Fetch all roles |
| POST | `/AUTHENTICATION/auth/getRole/{id}` | `apiServices/RoleServices/RoleServices.js` | Fetch role by ID |
| POST | `/AUTHENTICATION/auth/createRole` | `apiServices/RoleServices/RoleServices.js` | Create new role |
| POST | `/AUTHENTICATION/auth/deleteRole/{id}` | `apiServices/RoleServices/RoleServices.js` | Delete role |

### Permission Management
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/AUTHENTICATION/auth/getPermissions` | `apiServices/PermissionServices/PermissionServices.js` | Fetch all permissions |
| POST | `/AUTHENTICATION/auth/getPermission/{id}` | `apiServices/PermissionServices/PermissionServices.js` | Fetch permission by ID |
| POST | `/AUTHENTICATION/auth/createPermission` | `apiServices/PermissionServices/PermissionServices.js` | Create permission |
| POST | `/AUTHENTICATION/auth/deletePermission/{id}` | `apiServices/PermissionServices/PermissionServices.js` | Delete permission |

**AUTHENTICATION Total: 16 endpoints**

---

## FREESWITCHREST Gateway (`/FREESWITCHREST/`)

### Partner Management
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/FREESWITCHREST/partner/get-partners` | `apiServices/PartnerServices/PartnerServices.js` | Fetch all partners |
| POST | `/FREESWITCHREST/partner/get-partner` | `apiServices/PartnerServices/PartnerServices.js` | Fetch partner by ID |
| POST | `/FREESWITCHREST/partner/create-partner` | `apiServices/PartnerServices/PartnerServices.js` | Create partner |
| POST | `/FREESWITCHREST/partner/update-partner` | `apiServices/PartnerServices/PartnerServices.js` | Update partner |
| POST | `/FREESWITCHREST/partner/delete-partner` | `apiServices/PartnerServices/PartnerServices.js` | Delete partner |
| POST | `/FREESWITCHREST/partner/upload-multiple` | `views/services/Partner&Routes/Distributors/KYC.jsx` (inline, hardcoded URL) | Upload multiple KYC documents |

### Partner Prefix Management
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/FREESWITCHREST/get-partner-prefix-by-email` | `apiServices/PartnerPrefixServices/PartnerPrefix.js` | Get partner prefix by email |
| POST | `/FREESWITCHREST/get-partner-prefixes` | `apiServices/PartnerPrefixServices/PartnerPrefix.js` | Get all partner prefixes |
| POST | `/FREESWITCHREST/create-partner-prefix` | `apiServices/PartnerPrefixServices/PartnerPrefix.js` | Create partner prefix |
| POST | `/FREESWITCHREST/get-partner-prefix-by-id-partner` | `apiServices/PartnerPrefixServices/PartnerPrefix.js` | Get prefix by partner ID |
| POST | `/FREESWITCHREST/delete-partner-prefix` | `apiServices/PartnerPrefixServices/PartnerPrefix.js` | Delete partner prefix |

### Retail Partner Management
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/FREESWITCHREST/get-retail-partners` | `apiServices/RetailPartner/RetailPartnerServices.js` | Fetch all retail partners |
| POST | `/FREESWITCHREST/create-retail-partner` | `apiServices/RetailPartner/RetailPartnerServices.js` | Create retail partner |
| POST | `/FREESWITCHREST/delete-retail-partner` | `apiServices/RetailPartner/RetailPartnerServices.js` | Delete retail partner |
| POST | `/FREESWITCHREST/update-retail-partner` | `apiServices/RetailPartner/RetailPartnerServices.js` | Update retail partner |
| POST | `/FREESWITCHREST/create-retail-partner-from-file` | `apiServices/RetailPartner/RetailPartnerServices.js` | Bulk import retail partners from file |
| POST | `/FREESWITCHREST/get-retail-partner` | `apiServices/RetailPartner/RetailPartnerServices.js` | Get retail partner by partner ID |

### Route Management (SMS Routes)
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/FREESWITCHREST/route/get-routes` | `apiServices/SmsRouteService/SmsRouteService.js` | Fetch all routes |
| POST | `/FREESWITCHREST/route/get-route` | `apiServices/SmsRouteService/SmsRouteService.js` | Fetch route by ID |
| POST | `/FREESWITCHREST/route/create-route` | `apiServices/SmsRouteService/SmsRouteService.js` | Create route |
| POST | `/FREESWITCHREST/route/update-route` | `apiServices/SmsRouteService/SmsRouteService.js` | Update route |
| POST | `/FREESWITCHREST/route/delete-route` | `apiServices/SmsRouteService/SmsRouteService.js` | Delete route |
| POST | `/FREESWITCHREST/route/get-route-by-id-partner` | `apiServices/SmsRouteService/SmsRouteService.js` | Get routes by partner ID |

### Dial Plan Management
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/FREESWITCHREST/create-dialplan` | `apiServices/DialPlanServices/DialPlanServices.js` | Create dial plan |
| POST | `/FREESWITCHREST/update-dialplan` | `apiServices/DialPlanServices/DialPlanServices.js` | Update dial plan |
| POST | `/FREESWITCHREST/get-dialplans` | `apiServices/DialPlanServices/DialPlanServices.js` | Get all dial plans |
| POST | `/FREESWITCHREST/get-dialplan-by-id` | `apiServices/DialPlanServices/DialPlanServices.js` | Get dial plan by ID |
| POST | `/FREESWITCHREST/delete-dialplan` | `apiServices/DialPlanServices/DialPlanServices.js` | Delete dial plan |
| POST | `/FREESWITCHREST/create-dialplan-prefix` | `apiServices/DialPlanServices/DialPlanServices.js` | Create dial plan prefix |
| POST | `/FREESWITCHREST/update-dialplan-prefix` | `apiServices/DialPlanServices/DialPlanServices.js` | Update dial plan prefix |
| POST | `/FREESWITCHREST/get-dialplan-prefixes` | `apiServices/DialPlanServices/DialPlanServices.js` | Get all dial plan prefixes |
| POST | `/FREESWITCHREST/get-dialplan-prefix` | `apiServices/DialPlanServices/DialPlanServices.js` | Get dial plan prefix by ID |
| POST | `/FREESWITCHREST/delete-dialplan-prefix` | `apiServices/DialPlanServices/DialPlanServices.js` | Delete dial plan prefix |
| POST | `/FREESWITCHREST/dialplan-prefix/exists` | `apiServices/DialPlanServices/DialPlanServices.js` | Check if dial plan prefix exists |
| POST | `/FREESWITCHREST/add-route-to-dialplan` | `apiServices/DialPlanServices/DialPlanServices.js` | Add route to dial plan |
| POST | `/FREESWITCHREST/delete-dialplan-route` | `apiServices/DialPlanServices/DialPlanServices.js` | Delete route from dial plan |

### Call Source Management
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/FREESWITCHREST/create-callsrc` | `apiServices/CallSourceServices/CallSourceServices.js` | Create call source |
| POST | `/FREESWITCHREST/update-callsrc` | `apiServices/CallSourceServices/CallSourceServices.js` | Update call source |
| POST | `/FREESWITCHREST/get-callsrcs` | `apiServices/CallSourceServices/CallSourceServices.js` | Get all call sources |

### Digit Filter Plan Management
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/FREESWITCHREST/api/plans/list` | `apiServices/DigitFilterPlanServices/digitFilterPlanervices.js` | List digit filter plans |
| POST | `/FREESWITCHREST/api/plans/getById` | `apiServices/DigitFilterPlanServices/digitFilterPlanervices.js` | Get plan by ID |
| POST | `/FREESWITCHREST/api/plans/create` | `apiServices/DigitFilterPlanServices/digitFilterPlanervices.js` | Create plan |
| POST | `/FREESWITCHREST/api/plans/update` | `apiServices/DigitFilterPlanServices/digitFilterPlanervices.js` | Update plan |
| POST | `/FREESWITCHREST/api/plans/delete` | `apiServices/DigitFilterPlanServices/digitFilterPlanervices.js` | Delete plan |

### Digit Filter Rules (Plan Details)
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/FREESWITCHREST/api/digit-filter-rules/list` | `apiServices/DigitFilterPlanServices/DigitFilterPlanDetails.js` | List digit filter rules |
| POST | `/FREESWITCHREST/api/digit-filter-rules/getById` | `apiServices/DigitFilterPlanServices/DigitFilterPlanDetails.js` | Get rule by ID |
| POST | `/FREESWITCHREST/api/digit-filter-rules/create` | `apiServices/DigitFilterPlanServices/DigitFilterPlanDetails.js` | Create rule |
| POST | `/FREESWITCHREST/api/digit-filter-rules/update` | `apiServices/DigitFilterPlanServices/DigitFilterPlanDetails.js` | Update rule |
| POST | `/FREESWITCHREST/api/digit-filter-rules/delete` | `apiServices/DigitFilterPlanServices/DigitFilterPlanDetails.js` | Delete rule |

### Digit Filter Rule Actions (Digit Cut)
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/FREESWITCHREST/api/digit-filter-rules/actions/list` | `apiServices/DigitFilterPlanServices/DigitCut.js` | List digit cut actions |
| POST | `/FREESWITCHREST/api/digit-filter-rules/actions/getById` | `apiServices/DigitFilterPlanServices/DigitCut.js` | Get action by ID |
| POST | `/FREESWITCHREST/api/digit-filter-rules/actions/create` | `apiServices/DigitFilterPlanServices/DigitCut.js` | Create action |
| POST | `/FREESWITCHREST/api/digit-filter-rules/actions/update` | `apiServices/DigitFilterPlanServices/DigitCut.js` | Update action |
| POST | `/FREESWITCHREST/api/digit-filter-rules/actions/delete` | `apiServices/DigitFilterPlanServices/DigitCut.js` | Delete action |

### DID Pool Management
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/FREESWITCHREST/get-did-pools` | `apiServices/DIDPoolServices/DidPoolServices.js` | Get all DID pools |
| POST | `/FREESWITCHREST/get-did-pool` | `apiServices/DIDPoolServices/DidPoolServices.js` | Get DID pool by ID |
| POST | `/FREESWITCHREST/create-did-pool` | `apiServices/DIDPoolServices/DidPoolServices.js` | Create DID pool |
| POST | `/FREESWITCHREST/update-did-pool` | `apiServices/DIDPoolServices/DidPoolServices.js` | Update DID pool |
| POST | `/FREESWITCHREST/delete-did-pool` | `apiServices/DIDPoolServices/DidPoolServices.js` | Delete DID pool |
| POST | `/FREESWITCHREST/get-did-assignments` | `apiServices/DIDPoolServices/DidPoolServices.js` | Get all DID assignments |
| POST | `/FREESWITCHREST/get-did-assignment-by-didpool` | `apiServices/DIDPoolServices/DidPoolServices.js` | Get DID assignments by pool |
| POST | `/FREESWITCHREST/get-did-numbers-by-pool` | `apiServices/DIDPoolServices/DidPoolServices.js` | Get DID numbers by pool |
| POST | `/FREESWITCHREST/get-did-numbers` | `apiServices/DIDPoolServices/DidPoolServices.js` | Get all DID numbers |
| POST | `/FREESWITCHREST/create-did-assignment` | `apiServices/DIDPoolServices/DidPoolServices.js` | Create DID assignment |
| POST | `/FREESWITCHREST/get-did-assignment-by-partner-id` | `apiServices/DIDPoolServices/DidPoolServices.js` | Get DID assignment by partner |
| POST | `/FREESWITCHREST/get-did-number` | `apiServices/DIDPoolServices/DidPoolServices.js` | Get single DID number |
| POST | `/FREESWITCHREST/update-did-number` | `apiServices/DIDPoolServices/DidPoolServices.js` | Update DID number |
| POST | `/FREESWITCHREST/update-did-assignment` | `apiServices/DIDPoolServices/DidPoolServices.js` | Update DID assignment |
| POST | `/FREESWITCHREST/delete-did-assignment` | `apiServices/DIDPoolServices/DidPoolServices.js` | Delete DID assignment |
| POST | `/FREESWITCHREST/create-did-number` | `apiServices/DIDPoolServices/DidPoolServices.js` | Create DID number |
| POST | `/FREESWITCHREST/create-did-assign-from-csv` | `apiServices/DIDPoolServices/DidPoolServices.js` | Import DID assignments from CSV |
| POST | `/FREESWITCHREST/get-did-assignment-history` | `apiServices/DIDPoolServices/DidPoolServices.js` | Get DID assignment history |

### IP Authentication
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/FREESWITCHREST/api/ip-auth/get-ip-by-idauthuser` | `apiServices/UserServices/UserServices.js` | Get IP auth entries by user |
| POST | `/FREESWITCHREST/api/ip-auth/create` | `apiServices/UserServices/UserServices.js` | Create IP auth entry |
| POST | `/FREESWITCHREST/api/ip-auth/update` | `apiServices/UserServices/UserServices.js` | Update IP auth entry |
| POST | `/FREESWITCHREST/api/ip-auth/delete` | `apiServices/UserServices/UserServices.js` | Delete IP auth entry |

### SIP Registration
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/FREESWITCHREST/api/sofia/registrations` | `apiServices/UserServices/UserServices.js` | Get SIP registered users |
| POST | `/FREESWITCHREST/api/sofia/partner-wise-registrations` | `apiServices/UserServices/UserServices.js` | Get partner-wise SIP registrations |
| POST | `/FREESWITCHREST/getRegisteredUsers` | `apiServices/UserServices/UserServices.js` | Get registered users (hardcoded IP) |

### CDR / Call History
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/FREESWITCHREST/admin/getCallHistory` | `apiServices/CDRServices/CDRServices.js` | Admin call history |
| POST | `/FREESWITCHREST/user/getCallHistory` | `apiServices/CDRServices/CDRServices.js` | User call history |
| POST | `/FREESWITCHREST/get-partner-prefix-by-email` | `apiServices/CDRServices/CDRServices.js` | Get partner prefixes for CDR filter |
| POST | `/FREESWITCHREST/api/cdr/cdrByFilterForAdmin` | `apiServices/CDRServices/CDRServices.js` | Admin billing/CDR report |
| POST | `/FREESWITCHREST/api/cdr/cdrByFilterForUser` | `apiServices/CDRServices/CDRServices.js` | User billing/CDR report |
| POST | `/FREESWITCHREST/admin/downloadCallHistory` | `apiServices/CDRServices/CDRServices.js` | Export admin call history (blob) |
| POST | `/FREESWITCHREST/user/downloadCallHistory` | `apiServices/CDRServices/CDRServices.js` | Export user call history (blob) |
| POST | `/FREESWITCHREST/api/cdr/cdrByFilterExportForAdmin` | `apiServices/CDRServices/CDRServices.js` | Export admin CDR billing (blob) |
| POST | `/FREESWITCHREST/api/cdr/cdrByFilterExportForUser` | `apiServices/CDRServices/CDRServices.js` | Export user CDR billing (blob) |

### SIP Call Flow
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/FREESWITCHREST/sip/calls` | `views/services/Reports/CDRCallFlow/SipCallFlowVisualizer.jsx` (inline, hardcoded) | Fetch SIP call flow for visualization |

### Call Recordings
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/FREESWITCHREST/api/recordings/get-recordings` | `apiServices/CallRecordingsServices/CallRecordingsServices.js` | List call recordings |
| POST | `/FREESWITCHREST/api/recordings/stream` | `apiServices/CallRecordingsServices/CallRecordingsServices.js` | Stream/play recording (blob) |
| POST | `/FREESWITCHREST/api/recordings/auto-load` | `apiServices/CallRecordingsServices/CallRecordingsServices.js` | Auto-load recording (blob) |
| POST | `/FREESWITCHREST/api/recordings/download-zip` | `apiServices/CallRecordingsServices/CallRecordingsServices.js` | Download recordings as ZIP (blob) |

### Live Calls / Active Calls
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/FREESWITCHREST/concurrent-call-profile` | `apiServices/CallsServices/CallsServices.js` | Admin active calls profile |
| POST | `/FREESWITCHREST/concurrent-call-profile-partner` | `apiServices/CallsServices/CallsServices.js` | User active calls profile |
| POST | `/FREESWITCHREST/admin-live-calls-summary` | `apiServices/CallsServices/CallsServices.js` | Admin live calls summary |
| POST | `/FREESWITCHREST/user-live-calls-summary` | `apiServices/CallsServices/CallsServices.js` | User live calls summary |
| POST | `/FREESWITCHREST/concurrent-call-partner-wise` | `apiServices/CallsServices/CallsServices.js` | Partner-wise live calls |
| POST | `/FREESWITCHREST/sip-options-monitor/status` | `apiServices/CallsServices/CallsServices.js` | ICX/SIP options monitor status |
| POST | `/FREESWITCHREST/getConcurrentCall` | `apiServices/AdminDashboardServices/adminDashboardServices.js` | Get live concurrent call count |

### PBX Active Calls
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/FREESWITCHREST/api/v1/active-calls/list` | `apiServices/PbxActiveCallsServices/PbxActiveCallsServices.js` | List all active PBX calls (or by domain) |
| POST | `/FREESWITCHREST/api/v1/active-calls/details` | `apiServices/PbxActiveCallsServices/PbxActiveCallsServices.js` | Get PBX call details |
| POST | `/FREESWITCHREST/api/v1/active-calls/hangup` | `apiServices/PbxActiveCallsServices/PbxActiveCallsServices.js` | Hangup a PBX call |

### Admin Dashboard
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/FREESWITCHREST/admin/DashBoard/getIntervalWiseCall` | `apiServices/AdminDashboardServices/adminDashboardServices.js` | Admin call summary (interval-wise) |
| POST | `/FREESWITCHREST/user/DashBoard/getIntervalWiseCall` | `apiServices/AdminDashboardServices/adminDashboardServices.js` | User call summary (interval-wise) |
| POST | `/FREESWITCHREST/admin/DashBoard/getTotalCall` | `apiServices/AdminDashboardServices/adminDashboardServices.js` | Admin total calls |
| POST | `/FREESWITCHREST/admin/DashBoard/getOutgoingCall` | `apiServices/AdminDashboardServices/adminDashboardServices.js` | Admin outgoing calls |
| POST | `/FREESWITCHREST/admin/DashBoard/getIncomingCall` | `apiServices/AdminDashboardServices/adminDashboardServices.js` | Admin incoming calls |
| POST | `/FREESWITCHREST/admin/DashBoard/getMissedCall` | `apiServices/AdminDashboardServices/adminDashboardServices.js` | Admin missed calls |
| POST | `/FREESWITCHREST/user/DashBoard/getTotalCall` | `apiServices/AdminDashboardServices/adminDashboardServices.js` | User total calls |
| POST | `/FREESWITCHREST/user/DashBoard/getOutgoingCall` | `apiServices/AdminDashboardServices/adminDashboardServices.js` | User outgoing calls |
| POST | `/FREESWITCHREST/user/DashBoard/getIncomingCall` | `apiServices/AdminDashboardServices/adminDashboardServices.js` | User incoming calls |
| POST | `/FREESWITCHREST/user/DashBoard/getMissedCall` | `apiServices/AdminDashboardServices/adminDashboardServices.js` | User missed calls |
| POST | `/FREESWITCHREST/admin/DashBoard/system-info` | `apiServices/AdminDashboardServices/adminDashboardServices.js` | PBX system overview |
| POST | `/FREESWITCHREST/admin/DashBoard/top5PartnerCallCounts` | `apiServices/AdminDashboardServices/adminDashboardServices.js` | Top 5 partners by call count |

### Analytics / QoS Reports
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/FREESWITCHREST/analytics/graph-data` | `apiServices/AdminDashboardServices/adminDashboardServices.js` | Concurrent call graph data |
| POST | `/FREESWITCHREST/analytics/graph-data/csv` | `apiServices/AdminDashboardServices/adminDashboardServices.js` | Export graph data as CSV |
| POST | `/FREESWITCHREST/mcc-reports/max-call-periodically` | `apiServices/AdminDashboardServices/adminDashboardServices.js` | Max concurrent calls periodic report |
| POST | `/FREESWITCHREST/qos-reports/call-drop` | `apiServices/AdminDashboardServices/adminDashboardServices.js` | Call drop QoS report |
| POST | `/FREESWITCHREST/qos-reports/cssr` | `apiServices/AdminDashboardServices/adminDashboardServices.js` | CSSR QoS report |

### User Dashboard
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/FREESWITCHREST/getTotalCallForUser` | `apiServices/UserDashboardServices/userDashboardServices.js` | Get total calls for user |
| POST | `/FREESWITCHREST/user/DashBoard/getTopupBalanceForUser` | `apiServices/UserDashboardServices/userDashboardServices.js` | Get topup balance for user |
| POST | `/FREESWITCHREST/user/DashBoard/getBalance` | `apiServices/UserDashboardServices/userDashboardServices.js` | Get unified balance (prepaid/postpaid) |

### Package / Billing Management
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/FREESWITCHREST/package/create-package` | `apiServices/PackageServices/packageServices.js` | Create package |
| POST | `/FREESWITCHREST/package/get-packages` | `apiServices/PackageServices/packageServices.js` | Get all packages |
| POST | `/FREESWITCHREST/package/get-packageitems` | `apiServices/PackageServices/packageServices.js` | Get package items |
| POST | `/FREESWITCHREST/package/purchase-package` | `apiServices/PackageServices/packageServices.js` | Purchase package |
| POST | `/FREESWITCHREST/package/get-all-purchase` | `apiServices/PackageServices/packageServices.js` | Get all purchases |
| POST | `/FREESWITCHREST/package/get-all-purchase-partner-wise` | `apiServices/PackageServices/packageServices.js` | Get partner-wise purchase history |
| POST | `/FREESWITCHREST/package/get-all-topup` | `apiServices/PackageServices/packageServices.js` | Get all top-ups |
| POST | `/FREESWITCHREST/package/topup` | `apiServices/PackageServices/packageServices.js` | Process top-up |
| POST | `/FREESWITCHREST/package/create-package-items` | `apiServices/PackageServices/packageServices.js` | Create package items |
| POST | `/FREESWITCHREST/package/active-inactive` | `apiServices/PackageServices/packageServices.js` | Toggle package active/inactive |
| POST | `/FREESWITCHREST/package/update-onselect-Priority` | `apiServices/PackageServices/packageServices.js` | Update package priority order |
| POST | `/FREESWITCHREST/package/getPurchaseForPartner` | `apiServices/UserDashboardServices/userDashboardServices.js` | Get purchases for partner |
| POST | `/FREESWITCHREST/admin/DashBoard/get-partner-balances` | `apiServices/PackageServices/packageServices.js` | Admin: get partner balances |
| POST | `/FREESWITCHREST/admin/DashBoard/get-partner-balances-unified` | `apiServices/PackageServices/packageServices.js` | Admin: unified partner balances |
| POST | `/FREESWITCHREST/package/setup-postpaid-credit` | `apiServices/PackageServices/packageServices.js` | Setup postpaid credit |
| POST | `/FREESWITCHREST/package/update-postpaid-credit` | `apiServices/PackageServices/packageServices.js` | Update postpaid credit limit |
| POST | `/FREESWITCHREST/user/DashBoard/processPostpaidPayment` | `apiServices/PackageServices/packageServices.js` | Process postpaid payment |

### Summary Reports
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/FREESWITCHREST/api/summry-reports/dom-out-iptsp` | `apiServices/SummaryReportServices/SummaryReportServices.js` | Summary domestic outgoing report |
| POST | `/FREESWITCHREST/api/summry-reports/dom-in-iptsp` | `apiServices/SummaryReportServices/SummaryReportServices.js` | Summary domestic incoming report |

### MNP (Mobile Number Portability)
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/FREESWITCHREST/api/mnp_pull` | `apiServices/MnpServices/MnpServices.js` | Get MNP pull data |
| POST | `/FREESWITCHREST/api/mnp_pull/file-count` | `apiServices/MnpServices/MnpServices.js` | Get MNP pull file count |
| POST | `/FREESWITCHREST/api/mnp_pull/export-file-list` | `apiServices/MnpServices/MnpServices.js` | Export MNP pull file list |

### CCL Traffic Data
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/FREESWITCHREST/get-partner-details` | `apiServices/CCLDataServices/getCclIpTraffic.js` | Get CCL IP traffic (partner details) |
| POST | `/FREESWITCHREST/get-outgoing-calls` | `apiServices/CCLDataServices/getCclIpTraffic.js` | Get CCL outgoing IP calls |
| POST | `/FREESWITCHREST/get-btcl-calls` | `apiServices/CCLDataServices/getCclTdmTraffic.js` | Get CCL TDM/BTCL calls |

### Email
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/FREESWITCHREST/api/v1/email/send` | `apiServices/EmailServices/EmailService.js` | Send email |

### Audit Logs
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/FREESWITCHREST/get-audit-logs` | `apiServices/AuditLogsServices/AuditLogs.js` | Get audit logs |

### WebRTC Contacts
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/FREESWITCHREST/contact/get-contacts` | `apiServices/WebRtcServices/getWebRtcServices.js` | Get contacts |
| POST | `/FREESWITCHREST/contact/create-contact` | `apiServices/WebRtcServices/getWebRtcServices.js` | Create contact |
| POST | `/FREESWITCHREST/contact/delete-contact` | `apiServices/WebRtcServices/getWebRtcServices.js` | Delete contact |
| POST | `/FREESWITCHREST/contact/update-contact` | `apiServices/WebRtcServices/getWebRtcServices.js` | Update contact |
| POST | `/FREESWITCHREST/get-call-history` | `apiServices/WebRtcServices/getWebRtcServices.js` | Get WebRTC call history |

### Voice Broadcast
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/FREESWITCHREST/api/broadcast/play-audio` | `views/services/VoiceBroadcastFrontend/.../SendVoiceBroadcast.jsx` (inline, hardcoded) | Upload and send voice broadcast audio |

**FREESWITCHREST Total: ~135 endpoints**

---

## SMSREST Gateway (`/SMSREST/`)

### SMS Sending
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/SMSREST/SmsTask/sendSms` | `apiServices/smsApiServices/SmsTaskServices/SmsTaskServices.js` | Send SMS |
| GET | `/SMSREST/api/v2/promo/sendSmsViaGet` | `views/services/SMSFrontend/Client/SMSTask/SendSMS/SendSMS.jsx` (inline, hardcoded URL) | Send promo SMS via GET with API key auth |

### Campaign Management
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/SMSREST/campaign/save-campaign` | `apiServices/smsApiServices/CampaignsServices/CampaignsService.js` | Create campaign |
| POST | `/SMSREST/campaign/get-campaigns` | `apiServices/smsApiServices/CampaignsServices/CampaignsService.js` | Get campaigns |
| POST | `/SMSREST/campaign/enableCampaign` | `apiServices/smsApiServices/CampaignsServices/CampaignsService.js` | Enable campaign |
| POST | `/SMSREST/campaign/disableCampaign` | `apiServices/smsApiServices/CampaignsServices/CampaignsService.js` | Disable campaign |

### Campaign Tasks
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/SMSREST/campaignTask/get-campaign-tasks` | `apiServices/smsApiServices/SMSDashboard/CampaignTasksServices.js` | Get campaign tasks |

### SMS Dashboard
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/SMSREST/api/admin/dashboard/sms-and-campaign-summary` | `apiServices/smsApiServices/SMSDashboard/smsAndCampaignSummary.js` | SMS and campaign summary |
| POST | `/SMSREST/api/admin/dashboard/get-top-5-partner-sms-count` | `apiServices/smsApiServices/SMSDashboard/smsAndCampaignSummary.js` | Top 5 partners by SMS count |

### Contact Group Management
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/SMSREST/contact-group/create` | `apiServices/smsApiServices/ContactGroupServices/contactGroupServices.js` | Create contact group |
| POST | `/SMSREST/contact-group/getAll` | `apiServices/smsApiServices/ContactGroupServices/contactGroupServices.js` | Get all contact groups |
| POST | `/SMSREST/contact-group/update` | `apiServices/smsApiServices/ContactGroupServices/contactGroupServices.js` | Update contact group |
| POST | `/SMSREST/contact-group/delete` | `apiServices/smsApiServices/ContactGroupServices/contactGroupServices.js` | Delete contact group |

### Contact Item Management
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/SMSREST/contact-item/create` | `apiServices/smsApiServices/ContactGroupServices/contactItemServices.js` | Create contact item |
| POST | `/SMSREST/contact-item/uploadContactItems` | `apiServices/smsApiServices/ContactGroupServices/contactItemServices.js` | Upload/import contacts |
| POST | `/SMSREST/contact-item/getAll` | `apiServices/smsApiServices/ContactGroupServices/contactItemServices.js` | Get all contact items |
| POST | `/SMSREST/contact-item/getContactItemByGroupId` | `apiServices/smsApiServices/ContactGroupServices/contactItemServices.js` | Get contacts by group ID |
| POST | `/SMSREST/contact-item/update` | `apiServices/smsApiServices/ContactGroupServices/contactItemServices.js` | Update contact item |
| POST | `/SMSREST/contact-item/delete` | `apiServices/smsApiServices/ContactGroupServices/contactItemServices.js` | Delete contact item |

### Forbidden Words Management
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/SMSREST/forbidden-group/getAll` | `apiServices/smsApiServices/ForbiddenServices/ForbiddenServices.js` | Get all forbidden word groups |
| POST | `/SMSREST/forbidden-group/create` | `apiServices/smsApiServices/ForbiddenServices/ForbiddenServices.js` | Create forbidden word group |
| POST | `/SMSREST/forbidden-group/update` | `apiServices/smsApiServices/ForbiddenServices/ForbiddenServices.js` | Update forbidden word group |
| POST | `/SMSREST/forbidden-group/delete` | `apiServices/smsApiServices/ForbiddenServices/ForbiddenServices.js` | Delete forbidden word group |
| POST | `/SMSREST/forbidden-word-list/getForbiddenWordByGroupId` | `apiServices/smsApiServices/ForbiddenServices/ForbiddenServices.js` | Get forbidden words by group ID |
| POST | `/SMSREST/forbidden-word-list/create` | `apiServices/smsApiServices/ForbiddenServices/ForbiddenServices.js` | Create forbidden word |
| POST | `/SMSREST/forbidden-word-list/update` | `apiServices/smsApiServices/ForbiddenServices/ForbiddenServices.js` | Update forbidden word |
| POST | `/SMSREST/forbidden-word-list/delete` | `apiServices/smsApiServices/ForbiddenServices/ForbiddenServices.js` | Delete forbidden word |

### SMS Queue Management
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/SMSREST/api/sms-queue/create` | `apiServices/smsApiServices/SmsQueue/smsQueue.js` | Create SMS queue |
| POST | `/SMSREST/api/sms-queue/getAll` | `apiServices/smsApiServices/SmsQueue/smsQueue.js` | Get all SMS queues |
| POST | `/SMSREST/api/sms-queue/update` | `apiServices/smsApiServices/SmsQueue/smsQueue.js` | Update SMS queue |
| POST | `/SMSREST/api/sms-queue/delete` | `apiServices/smsApiServices/SmsQueue/smsQueue.js` | Delete SMS queue |

**Note**: The SMS queue URL construction has a bug: `${root2}:/SMSREST/api/sms-queue/` produces a double colon (e.g. `http://localhost:8001:/SMSREST/...`). This likely only works when `root2` omits the port.

### DND (Do Not Disturb) Group Management
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/SMSREST/dnd-group/create` | `apiServices/smsApiServices/DND/dndServicesServices.js` | Create DND group |
| POST | `/SMSREST/dnd-group/getAll` | `apiServices/smsApiServices/DND/dndServicesServices.js` | Get all DND groups |
| POST | `/SMSREST/dnd-group/update` | `apiServices/smsApiServices/DND/dndServicesServices.js` | Update DND group |
| POST | `/SMSREST/dnd-group/delete` | `apiServices/smsApiServices/DND/dndServicesServices.js` | Delete DND group |

### DND List Management
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/SMSREST/dnd-list/create` | `apiServices/smsApiServices/DND/dndListServices.js` | Create DND list entry |
| POST | `/SMSREST/dnd-list/getAll` | `apiServices/smsApiServices/DND/dndListServices.js` | Get all DND list entries |
| POST | `/SMSREST/dnd-list/getByGroup` | `apiServices/smsApiServices/DND/dndListServices.js` | Get DND entries by group |
| POST | `/SMSREST/dnd-list/update` | `apiServices/smsApiServices/DND/dndListServices.js` | Update DND list entry |
| POST | `/SMSREST/dnd-list/delete` | `apiServices/smsApiServices/DND/dndListServices.js` | Delete DND list entry |

### SMS Rate Plan Management
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/SMSREST/api/rate-plan/search` | `apiServices/smsApiServices/RatePlanServices/RatePlanServices.js` | Search rate plans |
| POST | `/SMSREST/api/rate-plan/getAllRatePlanById` | `apiServices/smsApiServices/RatePlanServices/RatePlanServices.js` | Get rate plan by ID |
| POST | `/SMSREST/api/rate-plan/create` | `apiServices/smsApiServices/RatePlanServices/RatePlanServices.js` | Create rate plan |
| POST | `/SMSREST/api/rate-plan/update` | `apiServices/smsApiServices/RatePlanServices/RatePlanServices.js` | Update rate plan |
| POST | `/SMSREST/api/rate-plan/deleteRatePlan` | `apiServices/smsApiServices/RatePlanServices/RatePlanServices.js` | Delete rate plan |

### SMS Rate Management
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/SMSREST/rate/getAllRate` | `apiServices/smsApiServices/RatePlanServices/RatePlanServices.js` | Get all rates |
| POST | `/SMSREST/rate/search` | `apiServices/smsApiServices/RatePlanServices/RatePlanServices.js` | Search rates by rate plan ID |
| POST | `/SMSREST/rate/getRateByIdPartner` | `apiServices/smsApiServices/BalanceServices/BalanceServices.js` | Get rate/balance by partner ID |

### SMS Rate Task Management
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/SMSREST/api/ratetask/getAllRateTaskByTaskRef?idRatePlan={id}` | `apiServices/smsApiServices/RatePlanServices/RatePlanServices.js` | Get rate tasks by plan ID |
| POST | `/SMSREST/api/ratetask/finalizeRateTaskCommit` | `apiServices/smsApiServices/RatePlanServices/RatePlanServices.js` | Finalize rate task commit |
| POST | `/SMSREST/api/ratetask/insertDirect` | `apiServices/smsApiServices/RatePlanServices/RatePlanServices.js` | Insert rate directly |
| POST | `/SMSREST/api/ratetask/insertBulkDirect` | `apiServices/smsApiServices/RatePlanServices/RatePlanServices.js` | Bulk insert rates directly |
| POST | `/SMSREST/api/ratetask/createRateTask` | `apiServices/smsApiServices/RatePlanServices/RatePlanServices.js` | Create rate task |

### Rate Plan Assignment
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/SMSREST/api/rateplan-assignment/allRatePlanAssign` | `apiServices/smsApiServices/RatePlanServices/RateplanAssignmentServices.js` | Get all rate plan assignments |
| POST | `/SMSREST/api/rateplan-assignment/assign` | `apiServices/smsApiServices/RatePlanServices/RateplanAssignmentServices.js` | Assign rate plan |
| POST | `/SMSREST/api/rateplan-assignment/deletePlanAssign` | `apiServices/smsApiServices/RatePlanServices/RateplanAssignmentServices.js` | Delete rate plan assignment |

### SMS Billing Rules
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/SMSREST/api/jsonbillingrules/getAllRules` | `apiServices/smsApiServices/RatePlanServices/RateplanAssignmentServices.js` | Get all billing rules |

### Dropdown / Reference Data
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/SMSREST/api/drop-down/getServiceFamily` | `apiServices/smsApiServices/RatePlanServices/RatePlanServices.js` | Get service families |
| POST | `/SMSREST/api/drop-down/getAllBillingsSpan` | `apiServices/smsApiServices/RatePlanServices/RatePlanServices.js` | Get billing spans |
| POST | `/SMSREST/api/drop-down/getTimeZone` | `apiServices/smsApiServices/RatePlanServices/RatePlanServices.js` | Get time zones |
| POST | `/SMSREST/api/drop-down/getAllUom` | `apiServices/smsApiServices/RatePlanServices/RatePlanServices.js` | Get currencies/units |
| POST | `/SMSREST/api/drop-down/getAllServiceCategory` | `apiServices/smsApiServices/RatePlanServices/RatePlanServices.js` | Get service categories |
| POST | `/SMSREST/api/drop-down/getAllServiceSubCategory` | `apiServices/smsApiServices/RatePlanServices/RatePlanServices.js` | Get service sub-categories |
| POST | `/SMSREST/api/drop-down/getAllCountryCode` | `apiServices/smsApiServices/RatePlanServices/RatePlanServices.js` | Get country codes |
| POST | `/SMSREST/api/drop-down/getServiceGroup` | `apiServices/smsApiServices/RatePlanServices/RateplanAssignmentServices.js` | Get service groups |
| POST | `/SMSREST/api/drop-down/getPartnerType` | `apiServices/smsApiServices/RatePlanServices/RateplanAssignmentServices.js` | Get partner types |

### SMS Policy Management
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/SMSREST/api/policies/create` | `apiServices/smsApiServices/Policy/policyService.js` | Create policy |
| POST | `/SMSREST/api/policies/getAll` | `apiServices/smsApiServices/Policy/policyService.js` | Get all policies |

### Time Bands
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/SMSREST/api/timeBands/create` | `apiServices/smsApiServices/TimeBands/timeBandsService.js` | Create time band |
| POST | `/SMSREST/api/timeBands` | `apiServices/smsApiServices/TimeBands/timeBandsService.js` | Get time bands |

### SMS Error / Retry Configuration
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/SMSREST/api/smserrors/allsmserrors` | `apiServices/smsApiServices/SmsError/SmsErrorServices.js` | Get all SMS errors |
| POST | `/SMSREST/api/retry-intervals/create` | `apiServices/smsApiServices/SmsError/retryIntervalsServices.js` | Create retry interval |
| POST | `/SMSREST/api/retry-intervals` | `apiServices/smsApiServices/SmsError/retryIntervalsServices.js` | Get retry intervals |
| POST | `/SMSREST/api/retry-cause-codes/create` | `apiServices/smsApiServices/SmsError/retryCauseCodeServices.js` | Create retry cause code |
| POST | `/SMSREST/api/retry-cause-codes` | `apiServices/smsApiServices/SmsError/retryCauseCodeServices.js` | Get retry cause codes |

### Job Status Enums
| Method | Path | Service File | Purpose |
|--------|------|-------------|---------|
| POST | `/SMSREST/enum-job-status/findAllStatus` | `apiServices/smsApiServices/statusApiServices/statusServices.js` | Get all job statuses |
| POST | `/SMSREST/enum-job-status/findStatusById` | `apiServices/smsApiServices/statusApiServices/statusServices.js` | Get job status by ID |

**SMSREST Total: ~72 endpoints**

---

## WebSocket Connections

### 1. Live Calls WebSocket (Main)
- **URL**: `ws[s]://{root}/FREESWITCHREST/ws/live-calls?token={jwt}`
- **Service**: `src/services/WebSocketService.jsx` (singleton)
- **Hook**: `src/helpers/useWebSocketLiveData.js`
- **Consumer**: `src/hooks/useActiveCalls.js`
- **Subscription endpoints** (sent as JSON messages after connection):
  - `/admin-live-calls-summary` — Admin live call stats
  - `/user-live-calls-summary` — User live call stats (with `idPartner`)
  - `/concurrent-call-partner-wise` — Partner-wise concurrent calls
  - `/concurrent-call-profile` — Admin concurrent call profile
  - `/concurrent-call-profile-partner` — User concurrent call profile (with `idPartner`)

### 2. SMS Live Log WebSocket
- **URL**: `ws://45.251.228.3:9001/log` (hardcoded)
- **Files**:
  - `src/views/services/SMSFrontend/SMSDashboard/LiveSms.jsx`
  - `src/views/services/VoiceBroadcastFrontend/VoiceDashboard/LiveSms.jsx`
- **Purpose**: Real-time SMS delivery log stream

### 3. SMS WebSocket Manager
- **Service**: `src/views/services/SMSFrontend/SMSWebSocket/SMSWebSocketManager.js`
- **Client**: `src/views/services/SMSFrontend/SMSWebSocket/WebSocketClient.jsx`
- **Purpose**: Generic SMS WebSocket client for real-time SMS events (status, delivery)
- **URL**: Configurable, passed as constructor parameter

### 4. Janus WebRTC Gateway WebSocket
- **Service**: `src/views/dashboard/WebRtc/WebSocketClient.jsx` (singleton)
- **Manager**: `src/views/dashboard/WebRtc/WebSocketManager.js`
- **Protocol**: Janus WebRTC Gateway protocol
- **Purpose**: SIP phone integration via browser (register, call, hangup, accept, decline)
- **URL**: Configured externally (connects to Janus media server)

---

## External / Hardcoded API Calls (Not Through API Gateway)

These calls bypass the normal `{root}/GATEWAY/` pattern:

| Method | URL | File | Purpose |
|--------|-----|------|---------|
| POST | `https://103.95.96.98:/FREESWITCHREST/getRegisteredUsers` | `apiServices/UserServices/UserServices.js` | Get SIP registered users (hardcoded IP) |
| POST | `http://192.168.0.207:8080/api/payment/ssl/initiate/` | `views/services/Dashboard/Pages/SuperAdmin/DashboardBalance.jsx` | SSLCommerz payment initiation |
| POST | `http://192.168.0.206:5070/rate-task` | `apiServices/RateTaskServices/getRateTaskServices.js` | Voice rate task (hardcoded dev IP) |
| POST | `http://192.168.0.206:5070/new-task` | `apiServices/RateTaskServices/getRateTaskServices.js` | Create voice rate task (hardcoded dev IP) |
| GET | `https://iptsp.cosmocom.net:/SMSREST/api/v2/promo/sendSmsViaGet` | `views/services/SMSFrontend/Client/SMSTask/SendSMS/SendSMS.jsx` | Promo SMS via GET (hardcoded URL) |
| POST | `http://103.95.96.76:/FREESWITCHREST/api/broadcast/play-audio` | `views/services/VoiceBroadcastFrontend/.../SendVoiceBroadcast.jsx` | Voice broadcast audio upload (hardcoded IP) |
| POST | `http://localhost:/FREESWITCHREST/sip/calls` | `views/services/Reports/CDRCallFlow/SipCallFlowVisualizer.jsx` | SIP call flow data (hardcoded localhost, no port) |

---

## Summary

| Gateway | Endpoint Count |
|---------|---------------|
| AUTHENTICATION | 16 |
| FREESWITCHREST | ~135 |
| SMSREST | ~72 |
| WebSocket connections | 4 types |
| External/hardcoded | 7 |
| **Total** | **~230 endpoints** |
