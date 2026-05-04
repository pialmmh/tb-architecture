# FusionPBX Integration -- FreeSwitchREST

FreeSwitchREST is a Spring Boot service that wraps FusionPBX (an open-source FreeSWITCH GUI/management platform) through two integration channels: direct PostgreSQL access to FusionPBX's database, and REST API calls to FusionPBX's custom PHP REST endpoint.

---

## Connection Details

### PostgreSQL (FusionPBX Database) -- btcl profile

| Property | Value |
|---|---|
| URL | `jdbc:postgresql://114.130.145.82:5432/fusionpbx` |
| Username | `fusionpbx` |
| Password | `BgpmJUnPvVKdAnkGqusujG3jPxw` |
| Driver | `org.postgresql.Driver` |

Config source: `application-btcl.properties` lines 40-45.

### FusionPBX REST API -- btcl profile

| Property | Value |
|---|---|
| URL | `https://hippbx.btcliptelephony.gov.bd/app/rest_api/rest.php` |
| API Key UUID | `0c1ece42-31ce-4174-99e2-37e709fe348b` |
| API Key Secret | `KwmCmUnRePdJurcRh4sx` |

Config source: `application-btcl.properties` lines 48-53.

### Authentication Pattern (REST API)

All FusionPBX REST API service classes use HTTP Basic authentication:
```
Authorization: Basic base64(keyUuid + ":" + keySecret)
```
Some services (FusionPbxCallbackService, FusionPbxCallBroadcastService, FusionPbxCallCenterService) use only the key-uuid without the secret: `base64(keyUuid + ":")`.

All requests are HTTP POST to the single `rest.php` endpoint, with an `"action"` field in the JSON body to select the operation.

---

## Dual Datasource Architecture

FreeSwitchREST uses two independent JPA datasources in a single Spring Boot application:

### Primary: MySQL (Dynamic/Multi-tenant)

- **Config class:** `freeswitch.config.database.MySQLConfig`
- **Bean names:** `mysqlDataSource` (primary), `mysqlEntityManager`, `mysqlTransactionManager`
- **Entity packages:** `freeswitch.entity.mysqlentity`, `freeswitch.entity.webentity`
- **Repository packages:** `freeswitch.repository.mysqlrepository`
- **Multi-tenancy:** Uses `DynamicRoutingDataSource` (extends `AbstractRoutingDataSource`) to switch MySQL databases per request based on JWT token resolution via `DatabaseContextResolver` interceptor.
- **Purpose:** Stores application data -- partners, packages, billing, CDR (MySQL copy), DIDs, dialplans, routes, call sources, user auth.

### Secondary: PostgreSQL (FusionPBX)

- **Config class:** `freeswitch.config.database.PostgresConfig`
- **Bean names:** `postgresDataSource`, `postgresEntityManager`, `postgresTransactionManager`
- **Entity packages:** `freeswitch.entity.postgresentity`
- **Repository packages:** `freeswitch.repository.postgresrepository`
- **Dialect:** `org.hibernate.dialect.PostgreSQLDialect`
- **Purpose:** Direct read/write access to FusionPBX's own PostgreSQL database for domains, extensions, dialplans, IVR menus, recordings, conferences, CDR (XML CDR), gateways, users, groups.
- **Health check:** `PostgresConnectionHealthCheck` validates FusionPBX DB connectivity on application startup.

Services that write to FusionPBX PostgreSQL tables use `@Transactional("postgresTransactionManager")` to ensure the correct transaction manager.

---

## FusionPBX Entities (PostgreSQL)

All entities live in `freeswitch.entity.postgresentity`. Each maps to a standard FusionPBX PostgreSQL table:

| Entity Class | Table | Primary Key | Description |
|---|---|---|---|
| `VDomain` | `v_domains` | `domain_uuid` | Multi-tenant PBX domains |
| `VExtension` | `v_extensions` | `extension_uuid` | SIP extensions/user endpoints |
| `VGateway` | `v_gateways` | `gateway_uuid` | SIP gateways/trunks |
| `VDialplan` | `v_dialplans` | `dialplan_uuid` | Dialplan entries (inbound/outbound routes) |
| `VDialplanDetail` | `v_dialplan_details` | `dialplan_detail_uuid` | Dialplan condition/action details |
| `VDestination` | `v_destinations` | `destination_uuid` | Inbound DID destinations |
| `VIvrMenu` | `v_ivr_menus` | `ivr_menu_uuid` | IVR menu definitions |
| `VIvrMenuOption` | `v_ivr_menu_options` | `ivr_menu_option_uuid` | IVR menu DTMF options |
| `VConference` | `v_conferences` | `conference_uuid` | Conference rooms |
| `VRecording` | `v_recordings` | `recording_uuid` | Audio recording metadata |
| `VUser` | `v_users` | `user_uuid` | FusionPBX web/portal users |
| `VUserGroup` | `v_user_groups` | `user_group_uuid` | User-to-group assignments |
| `VGroup` | `v_groups` | `group_uuid` | Permission groups (admin, user, superadmin) |
| `VXmlCdr` | `v_xml_cdr` | `xml_cdr_uuid` | FusionPBX XML CDR records |

### PostgreSQL Repositories

Each entity has a Spring Data JPA repository in `freeswitch.repository.postgresrepository`:

| Repository | Entity | Notable Methods |
|---|---|---|
| `VDomainRepository` | VDomain | Standard CRUD |
| `VExtensionRepository` | VExtension | Standard CRUD, used by multiple services |
| `VGatewayRepository` | VGateway | Standard CRUD |
| `VDialplanRepository` | VDialplan | Filtered by `app_uuid` to distinguish inbound vs outbound routes |
| `VDialplanDetailRepository` | VDialplanDetail | Linked to dialplans |
| `VDestinationRepository` | VDestination | Standard CRUD |
| `VIvrMenuRepository` | VIvrMenu | Standard CRUD |
| `VIvrMenuOptionRepository` | VIvrMenuOption | Standard CRUD |
| `VConferenceRepository` | VConference | Standard CRUD |
| `VRecordingRepository` | VRecording | Standard CRUD |
| `VUserRepository` | VUser | Standard CRUD |
| `VUserGroupRepository` | VUserGroup | Standard CRUD |
| `VGroupRepository` | VGroup | Standard CRUD |
| `VXmlCdrRepository` | VXmlCdr | Complex filtering with `JpaSpecificationExecutor`, date ranges, domain/direction/status |
| `VXmlCdrRepositoryAdmin` | VXmlCdr | Admin-level CDR queries |
| `VXmlCdrSpecifications` | -- | Dynamic JPA Specification builder for CDR filters |

---

## FusionPBX REST API Client Services

All REST API service classes live in `freeswitch.service` and follow a common pattern:
- Inject `fusionpbx.rest.url`, `fusionpbx.rest.key-uuid`, `fusionpbx.rest.key-secret` from properties
- Use `RestTemplate` to POST JSON to `rest.php` with Basic auth
- Send `"action"` field in JSON body to select the operation
- Parse JSON response with `ObjectMapper`

### Service Classes and Their REST API Actions

| Service Class | REST API Actions | Description |
|---|---|---|
| **FusionPbxActiveCallService** | `active-call-list`, `active-call-details`, `active-call-hangup`, `active-call-transfer`, `active-call-hold`, `active-call-mute`, `active-call-conference` | Live call management via ESL through PHP |
| **FusionPbxCallbackService** | `callback-config-create`, `callback-config-list`, `callback-config-get`, `callback-config-update`, `callback-config-delete`, `callback-config-toggle`, `callback-queue-create`, `callback-queue-list`, `callback-queue-get`, `callback-queue-cancel`, `callback-queue-retry`, `callback-install` | Callback queue configuration and management |
| **FusionPbxCallBlockService** | `call-block-list`, `call-block-create`, `call-block-update`, `call-block-delete`, `call-block-details` | Call blocking rules |
| **FusionPbxCallBroadcastService** | `call-broadcast-list`, `call-broadcast-details`, `call-broadcast-create`, `call-broadcast-update`, `call-broadcast-delete`, `call-broadcast-start`, `call-broadcast-stop`, `call-broadcast-upload-leads` | Voice call broadcast campaigns |
| **FusionPbxCallCenterService** | `call-center-queue-list`, `call-center-queue-details`, `call-center-queue-status`, `call-center-queue-live`, `call-center-queue-create`, `call-center-queue-update`, `call-center-queue-delete`, `call-center-agent-list`, `call-center-agent-details`, `call-center-agent-status`, `call-center-agent-set-status`, `call-center-agent-set-state`, `call-center-agent-create`, `call-center-agent-update`, `call-center-agent-delete`, `agent-stats`, `call-center-tier-list`, `call-center-tier-add`, `call-center-tier-remove`, `call-center-eavesdrop` | Full call center with queues, agents, tiers, eavesdrop |
| **FusionPbxCallForwardService** | `call-forward-list`, `call-forward-update`, `call-forward-toggle-dnd`, `call-forward-toggle-forward` | Call forwarding/DND settings per extension |
| **FusionPbxCallPermissionsService** | `call-permissions-list`, `call-permissions-update` | toll_allow permissions per extension |
| **FusionPbxCallRecordingService** | `call-recording-list`, `call-recording-details`, `call-recording-delete`, `call-recording-by-call-uuid`, `call-recording-by-sip-call-id`, `call-recording-bulk-download`, `call-recording-download`, `call-recording-bulk-info`, `call-recording-stream` | Call recording retrieval, download (including base64), bulk zip |
| **FusionPbxConferenceService** | `conference-create`, `conference-update`, `conference-delete`, `conference-toggle` | Conference CRUD with dialplan+cache management |
| **FusionPbxContactService** | `contact-list`, `contact-details`, `contact-create`, `contact-update`, `contact-delete`, `contact-list-by-extension` | PBX contact management |
| **FusionPbxDestinationService** | `destination-list`, `destination-details`, `destination-create`, `destination-update`, `destination-delete` | Inbound DID destination management |
| **FusionPbxGatewayService** | `gateway-list`, `gateway-details`, `gateway-create`, `gateway-update`, `gateway-delete` | SIP gateway/trunk CRUD with profile/transport metadata |
| **FusionPbxIvrService** | `ivr-list`, `ivr-details`, `ivr-create`, `ivr-update`, `ivr-delete` | IVR menu CRUD via PHP (with dialplan+cache) |
| **FusionPbxOutboundRouteService** | `outbound-route-list`, `outbound-route-details`, `outbound-route-create`, `outbound-route-update`, `outbound-route-delete` | Outbound route/dialplan management |
| **FusionPbxRecordingService** | `recording-create`, `recording-delete`, `recording-download` | Audio recording file upload/download (IVR prompts, MOH, etc.) |
| **FusionPbxRegistrationService** | `registration-list`, `registration-count`, `registration-unregister` | SIP registration status via `sofia status` |
| **FusionPbxRingGroupService** | `ring-group-list`, `ring-group-details`, `ring-group-create`, `ring-group-update`, `ring-group-delete`, `ring-group-toggle`, `ring-group-add-destination`, `ring-group-delete-destination` | Ring group management |
| **FusionPbxUserService** | `user-list`, `user-details`, `user-create`, `user-update`, `user-delete`, `user-groups` | FusionPBX portal user CRUD |

---

## Feature Map

### 1. Domains (Multi-tenant PBX Instances)

| | |
|---|---|
| **Controller** | `DomainController` (`/api/v1/domains`) |
| **Endpoints** | `create`, `list`, `get-by-uuid`, `get-by-name`, `update`, `delete`, `toggle` |
| **Access method** | **PostgreSQL direct** via `DomainService` -> `VDomainRepository` |
| **Entity/table** | `VDomain` / `v_domains` |
| **Notes** | Also writes to MySQL `Route` table on domain creation. Uses `@Transactional("postgresTransactionManager")`. |

### 2. Extensions (SIP User Endpoints)

| | |
|---|---|
| **Controller** | `ExtensionController` (`/api/v1/extensions`) |
| **Endpoints** | `create`, `list`, `list-by-domain`, `get-by-uuid`, `get-by-number`, `update`, `delete`, `toggle` |
| **Access method** | **Both** -- PostgreSQL direct for CRUD + REST API for creating associated FusionPBX user |
| **Entity/table** | `VExtension` / `v_extensions` |
| **Additional controller** | `VExtensionController` (`/vextension`) -- paginated list via `VExtensionService` |
| **Notes** | `ExtensionService` writes to `v_extensions` directly and optionally calls FusionPBX REST API. Also references `VDomainRepository`. |

### 3. Gateways/Trunks (SIP Connections)

| | |
|---|---|
| **Controller** | `GatewayController` (`/api/v1/gateways`) |
| **Endpoints** | `list`, `list-by-domain`, `details`, `get-by-uuid`, `create`, `update`, `delete`, `toggle`, `profiles`, `transports` |
| **Access method** | **REST API** via `FusionPbxGatewayService` |
| **DTOs** | `FusionGatewayRequest`, `FusionGatewayResponse` |
| **Notes** | Create/update supports full SIP gateway parameters (proxy, register, codecs, transport, etc.). Includes SSL trust-all for HTTPS. |

### 4. IVR Menus

| | |
|---|---|
| **Controller (direct PG)** | `IvrMenuController` (`/api/v1/ivr-menus`) |
| **Endpoints** | `create`, `list`, `list-by-domain`, `get-by-uuid`, `get-by-extension`, `update`, `delete`, `toggle`, `add-option`, `delete-option` |
| **Access method** | **PostgreSQL direct** via `IvrMenuService` -> `VIvrMenuRepository`, `VIvrMenuOptionRepository` |
| **Entity/table** | `VIvrMenu` / `v_ivr_menus`, `VIvrMenuOption` / `v_ivr_menu_options` |
| **Controller (REST API)** | `FusionPbxIvrController` (`/api/v1/fusionpbx-ivr`) |
| **Endpoints** | `list-by-domain`, `get-by-uuid`, `create`, `update`, `delete` |
| **Access method** | **REST API** via `FusionPbxIvrService` |
| **Notes** | Two parallel controllers exist -- direct PG for low-level CRUD, REST API for operations needing dialplan generation and cache clearing. |

### 5. Ring Groups

| | |
|---|---|
| **Controller** | `RingGroupController` (`/api/v1/ring-groups`) |
| **Endpoints** | `list`, `list-by-domain`, `get-by-uuid`, `create`, `update`, `delete`, `toggle`, `add-destination`, `delete-destination` |
| **Access method** | **REST API** via `FusionPbxRingGroupService` |
| **Notes** | Supports ring-simultaneous, ring-sequential strategies with timeout, destination management. |

### 6. Conferences

| | |
|---|---|
| **Controller** | `ConferenceController` (`/api/v1/conferences`) |
| **Endpoints** | `create`, `list-by-domain`, `get-by-uuid`, `update`, `delete`, `toggle` |
| **Access method** | **Both** -- PostgreSQL read via `VConferenceRepository` + REST API write via `FusionPbxConferenceService` |
| **Entity/table** | `VConference` / `v_conferences` |
| **Notes** | List/get-by-uuid reads from PostgreSQL directly. Create/update/delete/toggle goes through REST API to properly manage dialplans and clear FusionPBX cache. |

### 7. Call Center (Queues/Agents/Tiers)

| | |
|---|---|
| **Controller** | `CallCenterController` (`/api/call-center`) |
| **Queue endpoints** | `v1/queues/list`, `v1/queues/details`, `v1/queues/status`, `v1/queues/live`, `v1/queues/create`, `v1/queues/update`, `v1/queues/delete` |
| **Agent endpoints** | `v1/agents/list`, `v1/agents/details`, `v1/agents/status`, `v1/agents/set-status`, `v1/agents/set-state`, `v1/agents/create`, `v1/agents/update`, `v1/agents/delete`, `v1/agents/stats` |
| **Tier endpoints** | `v1/tiers/list`, `v1/tiers/add`, `v1/tiers/remove` |
| **Other** | `v1/eavesdrop` |
| **Access method** | **REST API** via `FusionPbxCallCenterService` |
| **Notes** | Full ACD with queue strategies, agent status/state control, tier levels, live monitoring, eavesdrop, agent KPI stats. |

### 8. Call Recordings

| | |
|---|---|
| **Controller** | `CallRecordingController` (`/api/recordings`) |
| **Endpoints** | `v1/list`, `v1/list-by-domain`, `v1/get-by-uuid`, `v1/delete`, `v1/get-by-call-uuid`, `v1/get-by-sip-call-id`, `v1/bulk-info`, `v1/bulk-download`, `v1/download` |
| **Access method** | **REST API** via `FusionPbxCallRecordingService` |
| **Notes** | Supports base64 download, SIP Call-ID lookup with fallback search, bulk zip download (Java-side zip creation), 200MB max string length for large recordings. |

### 9. Recordings (Audio Files -- IVR Prompts, MOH)

| | |
|---|---|
| **Controller** | `RecordingController` (`/api/v1/recordings`) |
| **Endpoints** | `create`, `list`, `list-by-domain`, `get-by-uuid`, `get-by-name`, `update`, `delete`, `download` |
| **Access method** | **Both** -- PostgreSQL direct via `RecordingService` -> `VRecordingRepository` + REST API via `FusionPbxRecordingService` for file operations |
| **Entity/table** | `VRecording` / `v_recordings` |
| **Notes** | Create uploads audio file (base64) via REST API and saves metadata to PostgreSQL. Delete removes both DB record and physical file via REST API. Download retrieves base64 from PHP. |

### 10. Inbound Routes (Destinations/Dialplan)

| | |
|---|---|
| **Controller (routes)** | `InboundRouteController` (`/api/v1/inbound-routes`) |
| **Endpoints** | `create`, `list-by-domain`, `get-by-uuid`, `update`, `delete`, `toggle` |
| **Access method** | **PostgreSQL direct** via `InboundRouteService` -> `VDialplanRepository`, `VDialplanDetailRepository` |
| **Entity/table** | `VDialplan` / `v_dialplans`, `VDialplanDetail` / `v_dialplan_details` |
| **Filter** | Uses `app_uuid = c03b422e-13a8-bd1b-e42b-b6b9b4d27ce4` (FusionPBX Inbound Routes app UUID) |
| **Controller (destinations)** | `DestinationController` (`/api/v1/destinations`) |
| **Endpoints** | `create`, `list`, `list-by-domain`, `list-by-type`, `get-by-uuid`, `get-by-number`, `update`, `delete`, `toggle` |
| **Access method** | **Both** -- PostgreSQL direct via `DestinationService` + REST API via `FusionPbxDestinationService` |
| **Entity/table** | `VDestination` / `v_destinations` |

### 11. Outbound Routes

| | |
|---|---|
| **Controller** | `OutboundRouteController` (`/api/v1/outbound-routes`) |
| **Endpoints** | `list`, `details`, `create`, `update`, `delete`, `toggle` |
| **Access method** | **REST API** via `FusionPbxOutboundRouteService` |
| **DTOs** | `FusionOutboundRouteRequest`, `FusionOutboundRouteResponse` |
| **Notes** | Also has PostgreSQL-direct service `OutboundRouteService` using `app_uuid = 8c914ec3-9fc0-8ab5-4cda-6c9288bdc9a3`. Controller primarily uses REST API path for proper dialplan generation. |

### 12. SIP Registration Status

| | |
|---|---|
| **Controller** | `RegistrationController` (`/api/v1/registrations`) |
| **Endpoints** | `list-by-domain`, `list-all`, `count`, `unregister`, `refresh` |
| **Access method** | **REST API** via `FusionPbxRegistrationService` |
| **DTO** | `FusionRegistrationResponse` (includes user, contact, network_ip, agent, status, ping info) |
| **Notes** | Reads from `sofia status` via PHP. Also has `SofiaStatusService` which runs local FreeSWITCH CLI commands. `list-by-domain` resolves domain name from `VDomainRepository` (PostgreSQL) then queries via REST API. |

### 13. CDR (Call Detail Records from FusionPBX XML CDR)

| | |
|---|---|
| **Controller** | `CdrApiController` (`/api/v1/cdr`) and `VXmlCdrController` |
| **Endpoints** | `list-by-domain`, `stats`, `get-by-uuid`, `hangup-causes` (CdrApiController); `get-call-history`, `downloadCallHistory` (VXmlCdrController) |
| **Access method** | **PostgreSQL direct** via `VXmlCdrRepository`, `VXmlCdrRepositoryAdmin`, `VXmlCdrSpecifications` |
| **Entity/table** | `VXmlCdr` / `v_xml_cdr` |
| **Notes** | Complex filtering by domain, direction, date range, caller number, duration, hangup cause. Supports Excel export. Also used by `DashboardService` and `DashboardServiceAdmin` for call statistics. |

### 14. FusionPBX Users (Portal/Web Users)

| | |
|---|---|
| **Controller** | `UserController` (`/api/v1/users`) |
| **Endpoints** | `list`, `details`, `create`, `update`, `delete`, `groups` |
| **Access method** | **REST API** via `FusionPbxUserService` |
| **DTOs** | `FusionUserRequest`, `FusionUserResponse`, `FusionApiRequest` |
| **Notes** | Manages FusionPBX web portal users (not SIP extensions). `groups` returns available permission groups. |

### 15. Active Calls (Live Call Management)

| | |
|---|---|
| **Controller** | `ActiveCallController` (`/api/v1/active-calls`) |
| **Endpoints** | `list`, `details`, `hangup`, `transfer`, `hold`, `mute`, `conference` |
| **Access method** | **REST API** via `FusionPbxActiveCallService` |
| **DTOs** | `ActiveCallResponse`, `ActiveCallDetailsResponse` |
| **Notes** | Real-time call control through FusionPBX PHP which proxies to FreeSWITCH ESL. Includes 3-way conference creation. |

### 16. Call Forwarding / DND

| | |
|---|---|
| **Controller** | `CallForwardController` (`/api/v1/call-forward`) |
| **Endpoints** | `list-by-domain`, `get-by-extension`, `update`, `toggle-dnd`, `toggle-forward-all` |
| **Access method** | **Both** -- PostgreSQL read via `VExtensionRepository` for listing + REST API via `FusionPbxCallForwardService` for updates (to clear FusionPBX cache) |
| **Notes** | Supports forward-all, forward-busy, forward-no-answer, forward-user-not-registered, DND. Updates go through PHP to clear XML cache. |

### 17. Call Block

| | |
|---|---|
| **Controller** | `CallBlockController` (`/api/v1/call-block`) |
| **Endpoints** | `list-by-domain`, `create`, `update`, `delete`, `get-by-uuid`, `toggle` |
| **Access method** | **REST API** via `FusionPbxCallBlockService` |

### 18. Call Broadcast

| | |
|---|---|
| **Controller** | `CallBroadcastController` (`/api/call-broadcast`) |
| **Endpoints** | `v1/list`, `v1/details`, `v1/create`, `v1/update`, `v1/delete`, `v1/start`, `v1/stop`, `v1/upload-leads` |
| **Access method** | **REST API** via `FusionPbxCallBroadcastService` |
| **Notes** | Voice broadcast campaigns with lead management. Start/stop controls campaign execution. |

### 19. Callback Queue

| | |
|---|---|
| **Controller** | `CallbackController` (`/api/v1/callback`) |
| **Endpoints** | `config/create`, `config/list`, `config/get`, `config/update`, `config/delete`, `config/toggle`, `queue/create`, `queue/list`, `queue/get`, `queue/cancel`, `queue/retry`, `install` |
| **Access method** | **REST API** via `FusionPbxCallbackService` |
| **Notes** | Customer callback queue management with configuration and queue item lifecycle. `install` creates the necessary DB tables in FusionPBX. |

### 20. Call Permissions (toll_allow)

| | |
|---|---|
| **Controller** | `CallPermissionsController` (`/api/v1/call-permissions`) |
| **Endpoints** | `list-by-domain`, `update` |
| **Access method** | **REST API** via `FusionPbxCallPermissionsService` |

### 21. FusionPBX Contacts

| | |
|---|---|
| **Controller** | `FusionPbxContactController` (`/api/v1/fusionpbx-contact`) |
| **Endpoints** | `list-by-domain`, `get-by-uuid`, `create`, `update`, `delete`, `list-by-extension` |
| **Access method** | **REST API** via `FusionPbxContactService` |

### 22. Database Health

| | |
|---|---|
| **Controller** | `DatabaseHealthController` (`/api/v1/health`) |
| **Endpoints** | `postgres` |
| **Access method** | **PostgreSQL direct** via `PostgresConnectionHealthCheck` |
| **Notes** | Returns FusionPBX PostgreSQL connection status. |

---

## DTO Classes for FusionPBX REST API

All in `freeswitch.dto.fusionpbx`:

| DTO | Used By |
|---|---|
| `FusionApiRequest` | General-purpose request (user CRUD) |
| `FusionUserRequest` / `FusionUserResponse` | FusionPbxUserService |
| `FusionGatewayRequest` / `FusionGatewayResponse` | FusionPbxGatewayService |
| `FusionDestinationRequest` / `FusionDestinationResponse` | FusionPbxDestinationService |
| `FusionOutboundRouteRequest` / `FusionOutboundRouteResponse` | FusionPbxOutboundRouteService |
| `FusionRegistrationResponse` | FusionPbxRegistrationService |
| `ActiveCallResponse` / `ActiveCallDetailsResponse` | FusionPbxActiveCallService |

---

## Access Method Summary

| Feature | PostgreSQL Direct | REST API | Why |
|---|---|---|---|
| Domains | YES | -- | Simple CRUD, no dialplan/cache |
| Extensions | YES | Partial (user creation) | Bulk of CRUD is direct, user link via REST |
| Gateways | -- | YES | Needs FreeSWITCH reload |
| IVR Menus | YES (low-level) | YES (with dialplan) | Two controllers: direct PG + REST for cache |
| Ring Groups | -- | YES | Needs dialplan generation |
| Conferences | YES (read) | YES (write) | Read direct, write via REST for dialplan |
| Call Center | -- | YES | Fully managed through FusionPBX |
| Call Recordings | -- | YES | Files on FusionPBX server filesystem |
| Recordings (audio) | YES (metadata) | YES (file ops) | Metadata in PG, files via REST |
| Inbound Routes | YES | -- | Direct dialplan manipulation |
| Outbound Routes | YES (available) | YES (primary) | REST preferred for proper dialplan gen |
| Destinations | YES | YES | Both paths available |
| Registrations | -- | YES | Live FreeSWITCH state via sofia |
| CDR | YES | -- | Historical data in v_xml_cdr |
| Users | -- | YES | FusionPBX user management |
| Active Calls | -- | YES | Real-time ESL via PHP |
| Call Forward | YES (read) | YES (write) | Read extensions PG, write via REST for cache |
| Call Block | -- | YES | FusionPBX managed |
| Call Broadcast | -- | YES | FusionPBX managed |
| Callback Queue | -- | YES | FusionPBX managed |
| Call Permissions | -- | YES | FusionPBX managed |
| Contacts | -- | YES | FusionPBX managed |

---

## Key Files

### Configuration
- `src/main/resources/application-btcl.properties` -- PostgreSQL + REST API connection config
- `src/main/java/freeswitch/config/database/PostgresConfig.java` -- Secondary datasource config
- `src/main/java/freeswitch/config/database/MySQLConfig.java` -- Primary datasource config
- `src/main/java/freeswitch/config/database/PostgresConnectionHealthCheck.java` -- Startup health check
- `src/main/java/freeswitch/config/database/DatabaseContextResolver.java` -- Request interceptor with full endpoint whitelist
- `src/main/java/freeswitch/config/database/DynamicRoutingDataSource.java` -- MySQL multi-tenant routing
- `src/main/java/freeswitch/config/database/DynamicDatabaseService.java` -- Database switching logic

### PostgreSQL Entities (`freeswitch.entity.postgresentity`)
- `VDomain.java`, `VExtension.java`, `VGateway.java`, `VDialplan.java`, `VDialplanDetail.java`, `VDestination.java`, `VIvrMenu.java`, `VIvrMenuOption.java`, `VConference.java`, `VRecording.java`, `VUser.java`, `VUserGroup.java`, `VGroup.java`, `VXmlCdr.java`

### PostgreSQL Repositories (`freeswitch.repository.postgresrepository`)
- `VDomainRepository.java`, `VExtensionRepository.java`, `VGatewayRepository.java`, `VDialplanRepository.java`, `VDialplanDetailRepository.java`, `VDestinationRepository.java`, `VIvrMenuRepository.java`, `VIvrMenuOptionRepository.java`, `VConferenceRepository.java`, `VRecordingRepository.java`, `VUserRepository.java`, `VUserGroupRepository.java`, `VGroupRepository.java`, `VXmlCdrRepository.java`, `VXmlCdrRepositoryAdmin.java`, `VXmlCdrSpecifications.java`

### FusionPBX REST API Services (`freeswitch.service`)
- `FusionPbxActiveCallService.java` -- Live call control
- `FusionPbxCallbackService.java` -- Callback queue
- `FusionPbxCallBlockService.java` -- Call blocking
- `FusionPbxCallBroadcastService.java` -- Voice broadcast
- `FusionPbxCallCenterService.java` -- Call center (queues, agents, tiers)
- `FusionPbxCallForwardService.java` -- Call forwarding/DND
- `FusionPbxCallPermissionsService.java` -- toll_allow
- `FusionPbxCallRecordingService.java` -- Call recording retrieval
- `FusionPbxConferenceService.java` -- Conference management
- `FusionPbxContactService.java` -- Contacts
- `FusionPbxDestinationService.java` -- DID destinations
- `FusionPbxGatewayService.java` -- SIP gateways
- `FusionPbxIvrService.java` -- IVR menus
- `FusionPbxOutboundRouteService.java` -- Outbound routes
- `FusionPbxRecordingService.java` -- Audio file management
- `FusionPbxRegistrationService.java` -- SIP registrations
- `FusionPbxRingGroupService.java` -- Ring groups
- `FusionPbxUserService.java` -- Portal users

### PostgreSQL-Direct Services (`freeswitch.service`)
- `DomainService.java` -- Domain CRUD
- `ExtensionService.java` -- Extension CRUD (hybrid: PG + REST)
- `InboundRouteService.java` -- Inbound route dialplan management
- `OutboundRouteService.java` -- Outbound route dialplan management
- `IvrMenuService.java` -- IVR menu direct CRUD
- `RecordingService.java` -- Recording metadata + file ops
- `DestinationService.java` -- Destination CRUD (hybrid: PG + REST)
- `VExtensionService.java` -- Paginated extension listing
- `VXmlCdrService.java` -- CDR queries and Excel export
- `CdrService.java` -- CDR with MySQL cross-reference
- `SofiaStatusService.java` -- SIP registration via local CLI + PG domain lookup
- `ActiveCallsService.java` -- Active calls with PG domain/extension enrichment
- `DashboardService.java` -- Dashboard stats from CDR
- `DashboardServiceAdmin.java` -- Admin dashboard with PG CDR

### FusionPBX DTOs (`freeswitch.dto.fusionpbx`)
- `FusionApiRequest.java`, `FusionUserRequest.java`, `FusionUserResponse.java`
- `FusionGatewayRequest.java`, `FusionGatewayResponse.java`
- `FusionDestinationRequest.java`, `FusionDestinationResponse.java`
- `FusionOutboundRouteRequest.java`, `FusionOutboundRouteResponse.java`
- `FusionRegistrationResponse.java`
- `ActiveCallResponse.java`, `ActiveCallDetailsResponse.java`

### Controllers Using FusionPBX
- `ActiveCallController.java`, `CallbackController.java`, `CallBlockController.java`, `CallBroadcastController.java`, `CallCenterController.java`, `CallForwardController.java`, `CallPermissionsController.java`, `CallRecordingController.java`, `ConferenceController.java`, `DestinationController.java`, `DomainController.java`, `ExtensionController.java`, `FusionPbxContactController.java`, `FusionPbxIvrController.java`, `GatewayController.java`, `InboundRouteController.java`, `IvrMenuController.java`, `OutboundRouteController.java`, `RecordingController.java`, `RegistrationController.java`, `RingGroupController.java`, `UserController.java`, `VExtensionController.java`, `VXmlCdrController.java`, `CdrApiController.java`, `DatabaseHealthController.java`

### Postman Collection
- `postman/FusionPBX_API.postman_collection.json`
