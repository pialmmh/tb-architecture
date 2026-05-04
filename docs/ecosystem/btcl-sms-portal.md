# BTCL SMS Portal -- Architecture Overview

## Summary

The BTCL SMS Portal (`btcl-sms-portal`) is a Next.js 15 customer-facing web application where BTCL enterprise customers can register, verify their identity (NID/trade license), and purchase four categories of services: Hosted PBX, Voice Broadcasting, Hosted Contact Center, and Bulk SMS. The portal handles the full lifecycle from multi-step registration with OTP/email verification and NID OCR, through package selection and checkout, to payment via SSLCommerz (Bangladesh payment gateway) or direct postpaid billing. It communicates with multiple backend Java REST services (RTC-Manager / FREESWITCHREST and an AUTHENTICATION service) for partner management, user auth, OTP, NID verification, and package purchase. The portal supports English/Bangla (bn) localization via next-intl and is deployed to an LXC container ("SoftSwitch") via PM2.

## Tech Stack

| Component | Technology |
|---|---|
| Framework | Next.js 15.5.2 (App Router) |
| Language | TypeScript 5 |
| React | React 18 |
| Styling | Tailwind CSS 3.4.6, tailwind-merge |
| UI Components | Headless UI 2.2.7, Lucide React icons |
| Forms | react-hook-form 7.62.0 |
| HTTP Client | axios 1.11.0, native fetch |
| Auth (local) | NextAuth 4.24.7 with Prisma adapter, JWT strategy, bcryptjs |
| Auth (backend) | JWT token from AUTHENTICATION service, stored in localStorage |
| ORM | Prisma 5.18.0 |
| Database | MySQL (via `DATABASE_URL` env var) |
| i18n | next-intl 3.17.2 (en, bn locales) |
| Payment | SSLCommerz (custom client in `src/lib/sslcommerz.ts`) |
| OCR | Tesseract.js 7.0.0 (NID card text extraction) |
| File Upload | uploadthing 6.13.2 |
| Toast | react-hot-toast 2.6.0 |
| JWT decode | jwt-decode 4.0.0 |
| Process Manager | PM2 (production) |

## Directory Structure

```
btcl-sms-portal/
  deploy.sh                       # LXC deployment script
  next.config.js                  # next-intl plugin, image config
  prisma/
    schema.prisma                 # MySQL schema (User, Account, Session, Document, Package, Order, SMSUsage)
    seed.ts                       # DB seeder
  src/
    app/
      layout.tsx                  # Root layout
      page.tsx                    # Root redirect
      api/
        auth/[...nextauth]/route.ts   # NextAuth route handler
        payment/
          initiate/route.ts           # Payment initiation (stub -- NOT IMPLEMENTED)
          success/route.ts            # SSLCommerz POST->GET redirect handler
          fail/route.ts               # Payment failure handler
          cancel/route.ts             # Payment cancellation handler
      [locale]/
        page.tsx                      # Home page (hero, services, pricing, testimonials)
        layout.tsx                    # Locale layout with AuthProvider
        about/page.tsx
        contact/page.tsx
        pricing/page.tsx
        register/page.tsx             # Multi-step registration (5 steps)
        postpaid-pending/page.tsx     # "Coming soon" page for postpaid
        services/
          page.tsx                    # Services index
          bulk-sms/page.tsx
          contact-center/page.tsx
          hosted-pbx/page.tsx
          voice-broadcast/page.tsx
        (auth)/                       # Auth route group
          layout.tsx
          login/page.tsx
          dashboard/page.tsx          # User dashboard (packages, purchases, documents)
      pg/                             # Payment gateway callback pages (outside locale)
        layout.tsx
        success/page.tsx              # Service-aware success page (PBX/VBS/generic)
        failed/page.tsx
        canceled/page.tsx
    components/
      checkout/
        CheckoutModal.tsx             # Main checkout flow with unified purchase
        CheckoutForm.tsx              # Billing details form
        OrderSummary.tsx              # Order summary with VAT calculation
        getPackageById.ts
      forms/
        FileUpload.tsx
        MultiStepForm.tsx
      layout/
        Header.tsx
        Footer.tsx
        LanguageToggle.tsx
      providers/
        SessionProvider.tsx
      toastProvider/
        ToastProvider.tsx
      ui/
        Button.tsx, Card.tsx, Input.tsx
      ProtectedRoute.tsx
    config/
      api.ts                          # Centralized API URL config and endpoints
    i18n.ts                           # next-intl request config
    lib/
      api-client/
        auth.ts                       # Login/register, token management, axios 401 interceptor
        partner.ts                    # Partner CRUD, OTP, email OTP, cross-service partner sync
        payment.ts                    # Unified purchase API call
      auth.ts                         # NextAuth config (credentials provider, JWT session)
      contexts/
        AuthContext.tsx               # Client-side JWT auth context with auto-logout
      nid-ocr.ts                      # NID card OCR via Tesseract.js
      prisma.ts                       # Prisma client singleton
      sslcommerz.ts                   # SSLCommerz payment client
      utils.ts
    messages/
      en.json                         # English translations
      bn.json                         # Bangla translations
    middleware.ts                      # next-intl middleware + payment POST->GET redirects
    types/
      next-auth.d.ts                  # NextAuth type extensions
      country-list.d.ts
```

## Key Components

### Pages/Routes

| Route | Description |
|---|---|
| `/` | Root redirect |
| `/[locale]` | Home page -- hero banner, service showcase (PBX, VBS, CC), pricing preview, testimonials |
| `/[locale]/about` | About BTCL |
| `/[locale]/contact` | Contact page |
| `/[locale]/pricing` | Pricing plans for all services |
| `/[locale]/register` | 5-step registration: (1) Company/Email/Phone + OTP verify, (2) Personal info + NID OCR, (3) Address/trade license/TIN docs, (4) Document upload, (5) Review & submit |
| `/[locale]/services` | Services index |
| `/[locale]/services/hosted-pbx` | Hosted PBX detail + pricing packages (Bronze/Silver/Gold) |
| `/[locale]/services/voice-broadcast` | Voice Broadcast detail + packages (Basic/Standard/Enterprise) |
| `/[locale]/services/contact-center` | Contact Center detail + per-agent pricing |
| `/[locale]/services/bulk-sms` | Bulk SMS detail |
| `/[locale]/login` | Login page |
| `/[locale]/dashboard` | Authenticated dashboard -- purchased packages, topup balance, partner documents, NID verification status |
| `/[locale]/postpaid-pending` | Placeholder "coming soon" for postpaid customers |
| `/pg/success` | Payment success callback (outside locale, handles PBX/VBS/generic flows) |
| `/pg/failed` | Payment failed callback |
| `/pg/canceled` | Payment canceled callback |

### API Routes (Next.js server-side)

| Route | Method | Purpose |
|---|---|---|
| `/api/auth/[...nextauth]` | GET/POST | NextAuth credential-based auth (Prisma adapter, JWT session) |
| `/api/payment/initiate` | POST | Payment initiation -- currently returns 501 (stub, not implemented; actual payment goes through backend unified purchase API) |
| `/api/payment/success` | POST/GET | SSLCommerz callback -- extracts tran_id/amount/status from POST form data, redirects to `/pg/success` |
| `/api/payment/fail` | POST | Redirects to `/pg/failed` |
| `/api/payment/cancel` | POST | Redirects to `/pg/canceled` |

**Note**: The `/api/payment/initiate` route is a stub. The actual payment flow uses the backend's unified purchase endpoint (`/api/payment/unified/purchase`) called directly from the client via `src/lib/api-client/payment.ts`.

### Auth

The portal has a **dual auth system**:

1. **NextAuth (server-side, Prisma-backed)**: Configured in `src/lib/auth.ts` with credentials provider. Uses JWT session strategy with Prisma adapter on MySQL. This appears to be a legacy/scaffolded auth that is not the primary auth mechanism.

2. **Backend JWT auth (primary)**: The actual auth flow uses the AUTHENTICATION service at `https://services.btcliptelephony.gov.bd/AUTHENTICATION`. Login returns a JWT token stored in `localStorage`. The `AuthContext` (`src/lib/contexts/AuthContext.tsx`) manages client-side auth state, decodes JWT expiry, sets up auto-logout timers, and checks token on tab visibility change. An axios interceptor in `src/lib/api-client/auth.ts` handles 401 responses globally.

### Database Schema (Prisma)

Models (MySQL, `prisma/schema.prisma`):

| Model | Key Fields | Purpose |
|---|---|---|
| `User` | email, name, phone, company, password, verificationStatus (PENDING/APPROVED/REJECTED/RESUBMISSION_REQUIRED) | Customer accounts |
| `Account` | userId, provider, providerAccountId | NextAuth OAuth accounts |
| `Session` | sessionToken, userId, expires | NextAuth sessions |
| `VerificationToken` | identifier, token, expires | Email verification tokens |
| `Document` | userId, type (NID/PASSPORT/TRADE_LICENSE/OTHER), fileUrl, status | Uploaded verification documents |
| `Package` | name, nameEn, nameBn, description, price, smsCount, validityDays, features (JSON) | Service packages |
| `Order` | userId, packageId, totalAmount, currency (BDT), status (PENDING/PAID/ACTIVATED/EXPIRED/CANCELLED/REFUNDED), transactionId, sslcommerzTranId, paymentData (JSON) | Purchase orders |
| `SMSUsage` | userId, orderId, smsCount, recipientCount, message, status, deliveredCount, failedCount, cost | SMS sending records |

**Note**: The Prisma schema is primarily scaffolded for local data. The actual partner/package/purchase data lives in the backend Java services (RTC-Manager databases).

### Payment Flow

Two payment paths based on `customerPrePaid` value from partner data:

**Path 1 -- Prepaid (customerPrePaid = 1): SSLCommerz Payment Gateway**
1. User clicks "Purchase" on a service package
2. `CheckoutModal` fetches partner data to determine `customerPrePaid`
3. For non-SMS services (PBX/VBS/CC), `ensurePartnerInService()` checks/creates the partner entity in the target service's backend
4. `unifiedPurchase()` calls `POST {ROOT_URL}/api/payment/unified/purchase` with package details, storeType (sms/pbx/vbs/cc), and `customerPrePaid=1`
5. Backend returns SSLCommerz `redirectUrl` / `GatewayPageURL`
6. Browser redirects to SSLCommerz payment page
7. SSLCommerz POSTs back to `/api/payment/success` (or fail/cancel)
8. Middleware converts POST to GET redirect to `/pg/success?tran_id=...&amount=...`
9. Success page reads `sessionStorage.pendingServiceProvision` to show service-specific success (PBX portal link, VBS portal link, or generic)

**Path 2 -- Postpaid (customerPrePaid = 2): Direct Purchase**
1. Same flow through step 3
2. `unifiedPurchase()` sends `customerPrePaid=2`
3. Backend processes purchase directly (no payment gateway), returns status
4. Success popup shown inline in `CheckoutModal`

**Exception**: Voice Broadcast always uses payment gateway (`effectivePrePaid = 1`) regardless of `customerPrePaid` value.

**VAT**: 15% VAT is added to all prices. Calculated client-side: `vat = Math.round(price * 0.15)`, `total = price + vat`.

### Registration Flow

5-step multi-step form in `src/app/[locale]/register/page.tsx`:

1. **Verification Info**: Company name, email (with email OTP), phone (with SMS OTP). Feature flags can disable OTP verification for testing.
2. **Personal Info**: Full name, alternate name, date of birth, NID number (10 or 17 digit), password. NID front/back image upload with Tesseract.js OCR that auto-fills name, NID number, and DOB.
3. **Other Info**: Customer type (prepaid/postpaid), address fields, trade license number, TIN number.
4. **Document Upload**: TIN certificate, NID front/back, VAT doc, trade license, photo, BIN certificate, SLA, BTRC registration, last tax return. Files sent via `addPartnerDetails()` as multipart FormData.
5. **Review & Submit**: Summary of all steps. On submit: creates partner via `POST /partner/create-partner`, then logs in via `/auth/login` to get JWT, then submits partner documents via `/partner/partner-documents`.

For postpaid customers (`customerType = 'postpaid'`), registration redirects to `/postpaid-pending` (under construction).

## External Connections

| Target | Base URL | Protocol | Purpose |
|---|---|---|---|
| Primary RTC-Manager (FREESWITCHREST) | `https://services.btcliptelephony.gov.bd/FREESWITCHREST` | REST/HTTPS | Partner CRUD, OTP send/verify, email OTP, package purchase, document upload, topup balance query |
| AUTHENTICATION service | `https://services.btcliptelephony.gov.bd/AUTHENTICATION` | REST/HTTPS | User login (`/auth/login`), registration (`/auth/register`), get user by email, edit user |
| NID verification service | `https://services.btcliptelephony.gov.bd/NID` | REST/HTTPS | NID verification (`/api/v1/nid/verify`) |
| VBS RTC-Manager | `https://vbs.btcliptelephony.gov.bd/FREESWITCHREST` | REST/HTTPS | Voice broadcast partner check/create, package purchase |
| PBX RTC-Manager | `https://vbs.btcliptelephony.gov.bd:4000/FREESWITCHREST` | REST/HTTPS (port 4000) | Hosted PBX partner check/create, package purchase |
| HCC RTC-Manager | `https://hcc.btcliptelephony.gov.bd/FREESWITCHREST` | REST/HTTPS | Contact center partner check/create |
| Unified Payment backend | `https://services.btcliptelephony.gov.bd/api/payment/unified/purchase` | REST/HTTPS | SSLCommerz initiation + direct purchase |
| SSLCommerz | `https://securepay.sslcommerz.com` (live) / `https://sandbox.sslcommerz.com` | HTTPS | Payment gateway API |
| Bulk SMS Portal | `https://a2psms.btcliptelephony.gov.bd/` | HTTPS (link only) | Dashboard links to external A2P SMS portal |
| PBX User Portal | `https://hippbx.btcliptelephony.gov.bd:5174/` | HTTPS (link only) | Post-purchase redirect for PBX customers |
| VBS Portal | `https://vbs.btcliptelephony.gov.bd/` | HTTPS (link only) | Post-purchase redirect for VBS customers |

### Backend API Endpoints Consumed

| Endpoint | Base | Method | Purpose |
|---|---|---|---|
| `/otp/send` | FREESWITCHREST | POST | Send phone OTP |
| `/otp/varify` | FREESWITCHREST | POST | Verify phone OTP (note: typo "varify" in backend) |
| `/otp/email/send` | FREESWITCHREST | POST | Send email OTP |
| `/otp/email/verify` | FREESWITCHREST | POST | Verify email OTP |
| `/auth/login` | AUTHENTICATION | POST | User login, returns JWT + roles + idPartner |
| `/auth/register` | AUTHENTICATION | POST | User registration |
| `/partner/create-partner` | FREESWITCHREST | POST | Create partner entity |
| `/partner/partner-documents` | FREESWITCHREST | POST/GET | Upload/retrieve partner documents (multipart) |
| `/partner/get-partner` | FREESWITCHREST | POST | Get partner by idPartner |
| `/partner/get-partner-extra` | FREESWITCHREST | POST | Get extended partner details |
| `/partner/get-partner-document` | FREESWITCHREST | POST | Get partner document |
| `/admin/DashBoard/partner/validate` | FREESWITCHREST | POST | Validate partner |
| `/package/getPurchaseForPartner` | FREESWITCHREST | POST | Get purchased packages for partner |
| `/package/purchase-package` | VBS/PBX FREESWITCHREST | POST | Purchase package in specific service |
| `/package/get-all-purchase-partner-wise` | FREESWITCHREST | POST | Get all purchases for partner |
| `/user/DashBoard/getTopupBalanceForUser` | FREESWITCHREST | POST | Get topup balance |
| `/getUserByEmail` | AUTHENTICATION | POST | Get user details by email |
| `/editUser` | AUTHENTICATION | POST | Update user (e.g., set pbxUuid) |
| `/api/v1/nid/verify` | NID service | POST | Verify NID number |
| `/api/payment/unified/purchase` | ROOT_URL | POST | Unified purchase (payment gateway or direct) |

## Data Flow

### Registration
```
User -> [Register Form] -> sendOtp/sendEmailOtp -> [FREESWITCHREST /otp/send, /otp/email/send]
     -> verifyOtp/verifyEmailOtp -> [FREESWITCHREST /otp/varify, /otp/email/verify]
     -> NID OCR (client-side Tesseract.js)
     -> createPartner -> [FREESWITCHREST /partner/create-partner] -> returns idPartner
     -> loginPartner -> [AUTHENTICATION /auth/login] -> returns JWT token
     -> addPartnerDetails -> [FREESWITCHREST /partner/partner-documents] (multipart with docs)
```

### Purchase (Prepaid)
```
User -> [CheckoutModal] -> getPartnerById -> [FREESWITCHREST /partner/get-partner]
     -> ensurePartnerInService -> [check target service, create if missing]
     -> unifiedPurchase -> [ROOT_URL /api/payment/unified/purchase] -> returns redirectUrl
     -> Browser redirect -> SSLCommerz payment page
     -> SSLCommerz POST -> /api/payment/success -> redirect -> /pg/success
     -> Success page shows service-specific portal link
```

### Dashboard
```
User -> [Dashboard] -> decode JWT -> get idPartner, email
     -> getTopupBalanceForUser -> [FREESWITCHREST /user/DashBoard/getTopupBalanceForUser]
     -> getPurchaseForPartner -> [FREESWITCHREST /package/getPurchaseForPartner]
     -> getPartnerById -> [FREESWITCHREST /partner/get-partner]
     -> getPartnerDocuments -> [FREESWITCHREST /partner/partner-documents]
```

## Configuration

### Environment Variables

| Variable | Purpose |
|---|---|
| `DATABASE_URL` | MySQL connection string for Prisma |
| `NEXTAUTH_SECRET` | NextAuth JWT signing secret |
| `NEXTAUTH_URL` | NextAuth base URL |
| `SSLCOMMERZ_STORE_ID` | SSLCommerz store identifier |
| `SSLCOMMERZ_STORE_PASSWORD` | SSLCommerz store password |
| `SSLCOMMERZ_IS_LIVE` | `true` for production, `false` for sandbox |
| `NEXT_PUBLIC_DEFAULT_LOCALE` | Default locale (en) |
| `NEXT_PUBLIC_SUPPORTED_LOCALES` | Supported locales (en,bn) |

### Feature Flags (`src/config/api.ts`)

| Flag | Default | Purpose |
|---|---|---|
| `OTP_VERIFICATION_ENABLED` | `true` | Toggle phone OTP verification during registration |
| `NID_VERIFICATION_ENABLED` | `true` | Toggle NID verification during registration |
| `PAYMENT_ENABLED` | `true` | Toggle SSLCommerz payment (when false, purchases succeed without payment) |

### API URL Configuration (`src/config/api.ts`)

All backend URLs are hardcoded in `src/config/api.ts` (not environment variables):

- `ROOT_URL`: `https://services.btcliptelephony.gov.bd`
- `VBS_BASE_URL`: `https://vbs.btcliptelephony.gov.bd/FREESWITCHREST`
- `PBX_BASE_URL`: `https://vbs.btcliptelephony.gov.bd:4000/FREESWITCHREST`
- `HCC_BASE_URL`: `https://hcc.btcliptelephony.gov.bd/FREESWITCHREST`
- `BULK_SMS_PORTAL_URL`: `https://a2psms.btcliptelephony.gov.bd/`

Commented-out localhost alternatives exist for local development.

### Package ID Mapping (hardcoded in `CheckoutModal.tsx`)

| Service | Package Name | Backend ID |
|---|---|---|
| hosted-pbx | Bronze | 9132 |
| hosted-pbx | Silver | 9133 |
| hosted-pbx | Gold | 9134 |
| voice-broadcast | Basic | 9135 |
| voice-broadcast | Standard | 9136 |
| voice-broadcast | Enterprise | 9137 |
| contact-center | Basic | 9140 |

## Deployment

### Production Deployment (`deploy.sh`)

Target: LXC container named "SoftSwitch" accessed via jump host `114.130.145.75:40001`.

Steps:
1. `npm run build` locally
2. Create `deploy.tar.gz` containing `.next/`, `public/`, `package.json`, `package-lock.json`, `next.config.js`
3. Upload to jump host via SCP (credentials from `.env.deploy`)
4. Push archive into LXC container via `lxc file push`
5. Inside container: extract to `/var/www/btcl-sms-portal`, `npm install --production`, start with PM2 as `btcl-portal`
6. PM2 configured for auto-startup

### Production Environment

- Deploy path: `/var/www/btcl-sms-portal`
- PM2 app name: `btcl-portal`
- Jump host: `114.130.145.75:40001` (user: `telcobright`)
- Container: `SoftSwitch`
- `.env` file with production DATABASE_URL and SSLCommerz credentials must be pre-configured on the container

## Integration Points with Other Projects

### RTC-Manager (FREESWITCHREST)

The portal is a frontend for multiple RTC-Manager instances. Each service (SMS, PBX, VBS, HCC) has its own RTC-Manager backend with the same REST API interface (`/FREESWITCHREST`). The portal:
- Creates partners in the primary service during registration
- Lazily creates partners in secondary services (PBX, VBS, HCC) on first purchase via `ensurePartnerInService()`
- Purchases packages by calling the service-specific `/package/purchase-package` endpoint

### AUTHENTICATION Service

Separate Java backend that manages user accounts, JWT tokens, and roles. Shared across all RTC-Manager instances. The portal uses it for login, registration, user lookup, and user edits.

### NID Verification Service

Backend service at `/NID/api/v1/nid/verify` for government NID verification. The portal also uses client-side Tesseract.js OCR as a UX convenience to pre-fill fields from NID card photos.

### Routesphere

No direct integration. Routesphere handles SMS routing/delivery. The portal's bulk SMS link (`a2psms.btcliptelephony.gov.bd`) points to a separate A2P SMS portal that likely interfaces with Routesphere.

### Payment Backend

The unified purchase endpoint at `{ROOT_URL}/api/payment/unified/purchase` is a backend service that handles both SSLCommerz gateway initiation and direct (postpaid) purchase. The `storeType` parameter (sms/pbx/vbs/cc) determines which backend service processes the purchase.

## Comparison: btcl-sms vs btcl-sms-portal

The older project at `/home/mustafa/telcobright-projects/btcl-sms/` is the original version. `btcl-sms-portal` is a significant evolution.

| Aspect | btcl-sms (old) | btcl-sms-portal (current) |
|---|---|---|
| **Next.js version** | 13.5.6 | 15.5.2 |
| **Services offered** | SMS-only (packages/pricing for SMS) | Multi-service: Hosted PBX, Voice Broadcast, Contact Center, Bulk SMS |
| **Backend integration** | None -- no `src/config/api.ts`, no `src/lib/api-client/` directory. Auth was NextAuth-only with Prisma DB | Full backend integration via axios/fetch to FREESWITCHREST, AUTHENTICATION, NID, payment APIs |
| **Auth mechanism** | NextAuth only (credentials + Prisma DB) | Dual: NextAuth scaffolded + primary JWT auth from AUTHENTICATION backend service |
| **Registration** | Basic NextAuth registration | 5-step multi-step form with phone OTP, email OTP, NID OCR (Tesseract.js), document upload, partner creation in backend |
| **Payment** | SSLCommerz stub (not connected to backend) | Full SSLCommerz integration via backend unified purchase API; supports prepaid (gateway) and postpaid (direct) |
| **Cross-service partner sync** | N/A | `ensurePartnerInService()` -- checks/creates partner in PBX/VBS/HCC before purchase |
| **Additional dependencies** | Minimal (no axios, no react-hook-form, no jwt-decode, no Tesseract, no Headless UI, no Lucide) | axios, react-hook-form, jwt-decode, Tesseract.js, Headless UI, Lucide React, country-list |
| **Routes** | `/[locale]/packages/`, `/[locale]/packages/[packageId]/purchase/`, `/[locale]/dashboard/`, `/[locale]/test/` | Service-specific pages (`/services/hosted-pbx`, `/services/voice-broadcast`, `/services/contact-center`, `/services/bulk-sms`), `/pg/*` payment callbacks, `/postpaid-pending` |
| **Dashboard** | Basic placeholder at `/[locale]/dashboard/` | Full dashboard with topup balance, purchased packages by service, partner documents viewer with image zoom, NID verification status |
| **Deployment** | No deploy script | `deploy.sh` -- build locally, deploy via SCP/LXC to SoftSwitch container |
| **Prisma schema** | Identical schema | Identical schema (not changed, but actual data now lives in backend Java services) |
| **Feature flags** | None | OTP_VERIFICATION_ENABLED, NID_VERIFICATION_ENABLED, PAYMENT_ENABLED |
| **i18n** | Same (en, bn) | Same (en, bn) |

Key takeaway: `btcl-sms` was a self-contained Next.js app with local Prisma DB for everything. `btcl-sms-portal` evolved into a frontend portal that delegates all business logic to backend Java services (RTC-Manager, AUTHENTICATION, Payment), retaining the Prisma schema as scaffolding but not using it as the primary data store for partners, packages, or purchases.
