# Config-Manager Architecture

## Summary

Config-manager is a Spring Boot centralized configuration service that serves as the configuration backbone for the routesphere ecosystem. It loads tenant configuration data from MySQL into an in-memory tenant tree, detects configuration changes via embedded Debezium CDC (MySQL binlog replication), and notifies routesphere-core of changes through Kafka and optionally Redis pub/sub. It exposes REST APIs for the tenant tree, CRM channel settings, Nacos-based runtime configuration, tenant provisioning, config reload, and version tracking.

**Source location:** `/home/mustafa/telcobright-projects/routesphere/config-manager/` (git submodule of parent routesphere repo)

## Tech Stack

| Component | Technology | Version |
|-----------|-----------|---------|
| Framework | Spring Boot | 3.2.5 |
| Java | JDK | 21 |
| Build | Maven | - |
| Database (primary) | MySQL | via mysql-connector-j |
| Database (secondary) | PostgreSQL | 42.7.3 |
| Messaging | Apache Kafka | spring-kafka 3.2.2 |
| Cache/Pub-Sub | Redis | spring-boot-starter-data-redis |
| CDC | Debezium Embedded | 2.5.0.Final |
| Schema Migration | Liquibase | 4.25.0 |
| HTTP Client | Spring WebFlux + OkHttp | 4.9.3 |
| Config Server | Nacos (external) | - |
| Connection Pool | HikariCP | 5.1.0 |
| Serialization | Jackson + Lombok | - |
| Excel | Apache POI | 5.2.2 |

## Directory Structure

```
config-manager/
  pom.xml
  config-manager-debezium-cdc.md              # Detailed CDC documentation
  doc/
    config-manager-overview.md                 # Quick reference doc
  src/main/
    java/
      freeswitch/
        ConfigManagerMain.java                 # @SpringBootApplication entry point
        config/
          AppConfig.java                       # General app configuration
          WebClientConfig.java                 # WebClient bean config
          database/
            DynamicRoutingDataSource.java       # AbstractRoutingDataSource impl
            MySQLConfig.java                    # MySQL datasource config
            PostgresConfig.java                 # Postgres datasource config
          dynamic/
            ConfigManager.java                  # Core: loads tenant tree, holds AtomicReference<Tenant>
            GlobalTenantRegistry.java           # Global partner/route/DID lookup maps
            core/
              AllCacheLoader.java               # Loads all caches into DynamicContext
              ConfigReloadListener.java         # Reload listener interface
              ConfigReloadNotifier.java         # Reload notification dispatch
              ConfigurationReloader.java        # Reload orchestrator
              DataLoader.java                   # Loads all entity types from DB
              TenantManager.java                # Builds tenant tree from admin + reseller DBs
            crm/
              CrmDataLoader.java                # Loads CRM entities (FB, WA, SMS settings)
              CrmGlobalRegistry.java            # CRM reverse-lookup maps
              CrmReloadListener.java            # CRM reload listener
              CrmSettingsManager.java           # CRM settings lifecycle manager
          kafka/
            ConfigReloader.java                 # Legacy Kafka consumer (DISABLED)
            KafkaConsumerConfig.java            # Kafka consumer beans
            KafkaUtil.java                      # Topic creation, message sending
            RecordParser.java                   # Kafka record parsing
          network/
            PrivateNetworkConfig.java           # Auto-binds HTTP to 10.10.x.x / 10.9.x.x
        controller/
          FsController.java                     # Tenant tree + reload + monitoring APIs
          UnifiedConfigController.java          # Combined MySQL + Nacos config API
          VersionController.java                # /api/version endpoint
          crm/
            CrmSettingsController.java          # CRM channel settings API
          nacos/
            NacosConfigController.java          # Nacos config proxy API
          tenant/
            TenantProvisioningController.java   # Tenant provisioning + schema migration API
        dto/                                     # Data transfer objects
        entity/tenant/                           # JPA entities for tenant config tables
        exception/                               # Global error handling
        repository/
          crmRepository/                         # CRM JPA repositories
          mysqlrepository/                       # Core entity repositories
          mysqlrepository/sms/                   # SMS-specific repositories
          postgresrepository/                    # Postgres repositories
        service/
          cdc/
            DebeziumCdcService.java             # Embedded Debezium engine (~1380 lines)
            DebeziumMysqlReplicationChecker.java # Auto-creates debezium_cdc MySQL user
          database/
            DynamicDatabaseService.java         # Dynamic DB switching, reseller DB discovery
          tenant/
            TenantConfigService.java            # Tenant config read service
            TenantSchemaService.java            # Liquibase migrations + provisioning
          nacos/
            NacosConfigService.java             # Nacos HTTP client + config parsing
            NacosHttpClient.java                # Raw HTTP client for Nacos
          configloader/
            CallSrcService.java                 # Call source loading
            DynamicContextService.java          # Dynamic context assembly
          sms/
            CampaignService.java
            EnumJobStatusService.java
            SmsQueueService.java
            TopicService.java
          ConfigPublishTracker.java             # Tracks published CDC event IDs
          ReloadEventTracker.java               # Reload frequency monitoring + stats
          SpringContext.java                    # Application context holder
          [... entity-specific services ...]
    resources/
      application.properties                    # Base config (port 7071, common settings)
      application-bdcom.properties              # Per-tenant: bdcom
      application-btcl.properties               # Per-tenant: btcl
      application-btcl_voice_broadcast.properties
      application-btcl_pbx.properties
      application-btcl_contact_center.properties
      application-l3venture.properties
      application-link3.properties
      application-ks_network.properties
      application-ccl76.properties
      application-ccl77.properties
      application-ccl98.properties
      application-ccl_76_voice_broadcasting.properties
      application-local.properties
      db/
        admin-init.sql                          # Admin DB bootstrap SQL
        changelog/
          changelog-master.xml                  # Liquibase master changelog
          001-initial-tenant-config-schema.xml  # Initial schema migration
      logback-spring.xml                        # Logging config
  tools/deploy/
    build.sh                                    # Maven build (requires JDK 21)
    remote-deploy-v2.sh                         # V2 SSH-inventory deploy script
    remote-setup.sh                             # Remote server setup
    configmanager.service                       # systemd unit file
    deploy-history.sh                           # Deploy history viewer
    tenant-conf-v2/                             # Per-tenant deploy targets (INI format)
      bdcom.conf
      btcl.conf
      btcl_voice_broadcast.conf
      btcl_pbx.conf
      btcl_contact_center.conf
      l3venture.conf
      link3.conf
      ks_network.conf
      ccl_76_voice_broadcasting.conf
  target/
    configmanager-1.0-SNAPSHOT.jar              # Build artifact
```

## Key Components

### ConfigManager (`freeswitch.config.dynamic.ConfigManager`)

Central orchestrator. Holds the root `Tenant` object in an `AtomicReference<Tenant>`. On startup (`@PostConstruct`), calls `loadConfigurations()` to build the tenant tree, then ensures SMS Kafka topics exist for all tenants. The `loadConfigurations()` method delegates to `TenantManager.buildTenantTree(adminDb)`.

### TenantManager (`freeswitch.config.dynamic.core.TenantManager`)

Builds the hierarchical tenant tree:
1. Creates a root `Tenant` from the admin database (`admin.db` property, e.g. `bdcom_sms`).
2. Calls `DynamicDatabaseService.getResellerDbs()` to discover child databases.
3. For each reseller DB, switches the datasource, loads a `DynamicContext`, and attaches it as a child of the root (or the appropriate parent based on underscore-delimited naming).

### DynamicDatabaseService (`freeswitch.service.database.DynamicDatabaseService`)

Handles dynamic database switching and reseller database discovery.

**Reseller DB discovery:** executes `SHOW DATABASES` and filters to databases starting with the `res_` prefix:
```java
public List<String> getResellerDbs() {
    return jdbcTemplate.queryForList("SHOW DATABASES;", String.class).stream()
            .filter(db -> db.startsWith("res_"))
            .collect(Collectors.toList());
}
```

This convention means any database named `res_*` on the same MySQL server is treated as a reseller/child tenant database. The tree hierarchy is derived from underscore segments: `res_foo` is a child of root, `res_foo_bar` is a child of `res_foo`.

Also maintains a `DID number -> DB name` lookup map across all databases.

### DataLoader (`freeswitch.config.dynamic.core.DataLoader`)

Loads all entity types from the current database into a `DynamicContext`:
- Partners, PartnerPrefixes, RetailPartners
- Routes, RouteMetadata
- CallSources, Dialplans, DialplanPrefixes, DialplanMappings
- DID Numbers, DID Assignments, DID Metadata
- RatePlans, Rates, RateAssignments
- Campaigns, Policies, SMS Queues
- DND Lists, Forbidden Words, Retry Cause Codes
- Package Purchases

### DebeziumCdcService (`freeswitch.service.cdc.DebeziumCdcService`)

Embedded Debezium engine (~1380 lines). Conditionally activated via `@ConditionalOnProperty(name="debezium.enabled", havingValue="true")`. Connects to MySQL as a replication slave, captures binlog events, filters by include/exclude patterns, batches changes, then on flush:
1. Calls `configManager.loadConfigurations()` (full tree reload)
2. Publishes notification JSON to Kafka topic (`config_event_loader_<tenant>`)
3. Optionally publishes to Redis channel (`config_event_loader_<tenant>`)

### PrivateNetworkConfig (`freeswitch.config.network.PrivateNetworkConfig`)

Auto-detects and binds the HTTP server to a private network interface (10.10.x.x preferred, 10.9.x.x fallback). This is required because MySQL users are restricted to private subnets. Activated via `network.private.auto-bind=true`.

### TenantSchemaService (`freeswitch.service.tenant.TenantSchemaService`)

Handles tenant database provisioning and Liquibase schema migrations. Two triggers:
- **On deploy (startup):** runs migrations on all registered tenants from `tenant_registry` table.
- **On Kafka event:** listens on `tenant_registration` topic; provisions new tenant database, runs migrations, seeds default config, notifies routesphere.

### CrmSettingsManager (`freeswitch.config.dynamic.crm.CrmSettingsManager`)

Loads CRM channel settings (Facebook, WhatsApp, SMS) from the admin database's `tenantinfo` table, iterates each active tenant, switches database, and populates a `CrmGlobalRegistry` with reverse-lookup maps (e.g., FB page ID to tenant DB name). Uses atomic swap for thread-safe reloads.

## REST Endpoints

### Tenant Tree API (`FsController`)

| Method | Path | Description |
|--------|------|-------------|
| POST | `/get-tenant-root` | Returns complete tenant tree from root |
| POST | `/get-specific-tenant-root?name=<tenant>` | Returns tenant tree (currently same as root) |
| POST | `/get-global-tenant-registry` | Returns GlobalTenantRegistry (partner IDs, route IPs, etc.) |
| POST | `/get-partner-vs-dbnames` | Returns partner ID to DB name mapping |

### Config Reload API (`FsController`)

| Method | Path | Description |
|--------|------|-------------|
| POST | `/reload` | Force manual config reload |
| POST | `/reload-status` | Last reload info (time, duration, trigger) |
| GET | `/reload-stats` | Reload frequency statistics |
| GET | `/reload-stats/tables?window=1h` | Per-table reload stats within time window |
| GET | `/reload-stats/hot?threshold=N` | Tables exceeding reload threshold |
| GET | `/reload-stats/table?name=<table>` | Timeline for a specific table |
| GET | `/reload-stats/config` | Reload monitor configuration |

### Unified Config API (`UnifiedConfigController`)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/v1/unified-config` | Combined MySQL entities + Nacos runtime config for all tenants |
| GET | `/api/v1/unified-config/health` | Health check (Nacos + MySQL) |

### Nacos Config API (`NacosConfigController`)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/config` | All tenant runtime configs from Nacos |
| GET | `/api/config/tenants` | List available tenant codes |
| GET | `/api/config/{tenantCode}` | Specific tenant config |
| GET | `/api/config/{tenantCode}/{dataId}` | Specific config file (parsed YAML) |
| GET | `/api/config/{tenantCode}/{dataId}/raw` | Raw YAML content |
| GET | `/api/config/health` | Nacos connection health |

### CRM Settings API (`CrmSettingsController`)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/crm/settings` | Complete CRM cache (tenants + reverse lookups) |
| POST | `/api/crm/settings/reload` | Reload CRM settings cache |

### Tenant Provisioning API (`TenantProvisioningController`)

| Method | Path | Description |
|--------|------|-------------|
| POST | `/api/v1/tenant/provision` | Provision new tenant (create DB, migrate, seed) |
| POST | `/api/v1/tenant/{tenantCode}/refresh` | Refresh existing tenant schema |
| POST | `/api/v1/tenant/migrate-all` | Run migrations on all registered tenants |
| GET | `/api/v1/tenant/{tenantCode}/config` | Get tenant configuration DTO |

### Version API (`VersionController`)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/version` | Build/deploy version info from `/opt/configmanager/version.txt` |

## External Connections

### MySQL

- Primary datastore for all tenant configuration entities
- Connection per tenant profile: `spring.datasource.url` in `application-<tenant>.properties`
- Dynamic database switching via `DynamicRoutingDataSource` (AbstractRoutingDataSource pattern)
- Reseller DBs discovered via `SHOW DATABASES` filtered by `res_` prefix
- Debezium CDC connects as replication slave (separate `debezium_cdc` user)

### Kafka (3-node KRaft cluster)

```
bootstrap-servers=10.10.199.20:9092,10.10.198.20:9092,10.10.197.20:9092
```

Config-manager is primarily a **Kafka producer**:
- Publishes CDC reload notifications to `config_event_loader_<tenant>` topic
- Creates SMS queue topics at startup (per-tenant suffix)
- Optionally forwards raw CDC events to `debezium_mysql_cdc` topic (proxy mode)
- Listens on `tenant_registration` topic for new tenant provisioning events

Default topic settings: `partitions=3, replication-factor=2`.

### Redis

- Optional pub/sub channel for CDC notifications (`config_event_loader_<tenant>`)
- Faster notification path than Kafka for latency-sensitive consumers
- Configurable per tenant: `debezium.notify.redis.enabled=true/false`
- Connection: `spring.data.redis.host` / `spring.data.redis.port` (typically `10.10.197.20:6379`)

### Nacos

- External configuration server at `localhost:7848`
- Provides runtime YAML configs (SMS, Voice, modules) organized by tenant namespaces
- Two modes: `single` (one tenant) or `multi` (all matching namespace prefix)

## Data Flow

### Startup Sequence

```
1. ConfigManagerMain (Spring Boot)
2. PrivateNetworkConfig: detect + bind to 10.10.x.x
3. MySQLConfig: create primary datasource
4. DynamicDatabaseService.loadDbVsDidNumberMap():
   - SHOW DATABASES -> filter res_* -> build DID-to-DB map
5. ConfigManager.init():
   - TenantManager.buildTenantTree(adminDb)
     - Load root tenant DynamicContext from admin DB
     - For each res_* DB: switch DB, load DynamicContext, attach as child
   - KafkaUtil.ensureSmsTopics() for all tenants
6. TenantSchemaService.onDeploy(): run Liquibase migrations on all registered tenants
7. CrmSettingsManager.init(): load CRM settings from all active tenants
8. DebeziumMysqlReplicationChecker: create debezium_cdc MySQL user (@Order(1))
9. DebeziumCdcService: start embedded Debezium engine (if enabled)
```

### CDC Change Detection Flow

```
MySQL binlog
  -> Debezium Engine (embedded, replication slave)
  -> Event filtering (database include, table include/exclude patterns)
  -> Batch collection (ConcurrentHashSet, pendingChanges)
  -> Fixed-rate flush (every 1s by default)
  -> configManager.loadConfigurations() [full tree rebuild]
  -> Dual-channel notification:
     +-- Kafka: config_event_loader_<tenant> topic
     +-- Redis: config_event_loader_<tenant> channel (optional)
```

### Routesphere-Core Consumer Side

Routesphere-core's `ConfigEventConsumer` listens on both Kafka and Redis channels, debounces reloads (default 3000ms), and triggers `TenantHierarchyInitializer.buildTenantFromConfigManager()` to refresh its local config.

## Configuration

### Base Config (`application.properties`)

```properties
server.port=7071
spring.profiles.active=bdcom                    # Default profile (overridden by deploy)
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
network.private.auto-bind=true                  # Binds to private IP, NOT 0.0.0.0
nacos.server-addr=localhost:7848
nacos.tenant.mode=multi
tenant.schema.auto-migrate=true
spring.liquibase.enabled=false                  # Programmatic execution via TenantSchemaService
reload.monitor.threshold-per-hour=60
```

### Per-Tenant Profile (`application-<tenant>.properties`)

Each tenant profile defines:

| Property | Example | Description |
|----------|---------|-------------|
| `admin.db` | `bdcom_sms` | Admin database name |
| `spring.datasource.url` | `jdbc:mysql://10.10.199.10:3306/${admin.db}` | JDBC URL |
| `datasource.url-base` | `jdbc:mysql://10.10.199.10:3306/` | Base URL for dynamic DB switching |
| `spring.kafka.bootstrap-servers` | `10.10.199.20:9092,...` | Kafka cluster |
| `debezium.enabled` | `true`/`false` | Enable embedded CDC |
| `debezium.database.include.list` | `bdcom_sms` | Databases to capture |
| `debezium.table.exclude.patterns` | `*cdr*,*campaign_task*,...` | Tables to skip |
| `debezium.notify.topic` | `config_event_loader` | Kafka notification topic |
| `debezium.notify.redis.enabled` | `true`/`false` | Redis pub/sub path |
| `spring.data.redis.host` | `10.10.197.20` | Redis host |
| `crm.settings.enabled` | `true`/`false` | Enable CRM settings loading |

### Tenant Configuration Summary

| Tenant | Profile | Admin DB | MySQL Host | CDC | Redis Notify | Deploy Server |
|--------|---------|----------|------------|-----|-------------|---------------|
| bdcom | bdcom | bdcom_sms | 10.10.199.10 | false | true | bdcom1 |
| btcl | btcl | btcl_sms | 127.0.0.1 | false | - | dell-sms-master |
| l3venture | l3venture | l3ventures_sms | 10.10.199.10 | **true** | false | l3venture1 |
| link3 | link3 | link3_sms | 10.10.199.10 | false | - | link3-1 |
| btcl_voice_broadcast | btcl_voice_broadcast | btcl_voicebroadcasting | 10.10.194.10 | **true** | - | sbc3 |
| btcl_pbx | btcl_pbx | (btcl_sms) | 10.10.194.10 | **true** | - | sbc1 |
| btcl_contact_center | btcl_contact_center | btcl_contact_center | 10.10.194.10 | false | - | sbc2 |
| ks_network | ks_network | ks_network_sms | 172.27.27.135 | false | - | ks-prod |

### CDC Notification JSON Format

```json
{
  "event": "config_reload",
  "changes": ["bdcom_sms.partner:UPDATE", "bdcom_sms.route:INSERT"],
  "changeCount": 2,
  "reloadDurationMs": 150,
  "eventId": "<uuid>",
  "timestamp": 1706900000000
}
```

## Deployment

### Build

```bash
cd /home/mustafa/telcobright-projects/routesphere/config-manager
bash tools/deploy/build.sh
```

Requires JDK 21. Produces `target/configmanager-1.0-SNAPSHOT.jar`.

### Deploy

```bash
bash tools/deploy/remote-deploy-v2.sh <tenant> <profile>
# Examples:
bash tools/deploy/remote-deploy-v2.sh bdcom dev
bash tools/deploy/remote-deploy-v2.sh btcl_voice_broadcast dev
bash tools/deploy/remote-deploy-v2.sh l3venture dev
```

Deploy steps:
1. Reads tenant config from `tools/deploy/tenant-conf-v2/<tenant>.conf` (INI format with `[dev]`/`[prod]` sections)
2. Resolves SSH connection from inventory at `routesphere-core/tools/ssh-automation/servers/<inventory_tenant>/`
3. Validates clean git working tree (refuses uncommitted code)
4. Auto-creates release tag on parent repo (`vYYYYMMDD-HHMM`)
5. Backs up current JAR on remote (keeps last 3)
6. Copies JAR, systemd service file, and setup script to remote
7. Runs setup script, writes `version.txt` and appends to `deploy-history.log`

### Tenant Deploy Config Format (`tenant-conf-v2/<tenant>.conf`)

```ini
[dev]
inventory_tenant = bdcom
server = bdcom1
description = BDCOM Development Server
debug_port = 5005
```

### Remote Paths (After Deploy)

| Item | Path |
|------|------|
| JAR | `/opt/configmanager/configmanager.jar` |
| Version | `/opt/configmanager/version.txt` |
| Log | `/var/log/configmanager/configmanager.log` |
| Env config | `/etc/configmanager/environment` |
| Service | `systemctl status configmanager` |
| Deploy history | `/opt/configmanager/deploy-history.log` |

### systemd Service

Runs as `configmanager` user, JVM flags: `-Xms1g -Xmx2g -XX:+UseG1GC`. Active Spring profile set via `TENANT_PROFILE` from `/etc/configmanager/environment`. Optional remote debug via `DEBUG_OPTS`.

## Integration Points

### With routesphere-core (Quarkus)

- **REST:** routesphere-core calls `POST /get-tenant-root` to fetch the full tenant tree at startup
- **REST:** routesphere-core calls `GET /api/crm/settings` for CRM channel settings
- **REST:** routesphere-core calls `GET /api/v1/unified-config` for combined config
- **Kafka:** config-manager publishes `config_event_loader_<tenant>` notifications; routesphere-core's `ConfigEventConsumer` listens and triggers local config rebuild
- **Redis:** optional fast-path notification for the same reload events
- **Version:** `GET /api/version` queryable from routesphere or monitoring

### With MySQL

- Reads all configuration entities (partners, routes, rates, DIDs, campaigns, policies, etc.)
- Connects as replication slave for CDC (Debezium)
- Auto-creates `debezium_cdc` MySQL user with REPLICATION privileges
- Runs Liquibase migrations on tenant databases

### With Kafka Cluster

- Produces CDC reload notifications
- Creates SMS queue topics at startup
- Consumes `tenant_registration` events for auto-provisioning

### With Nacos

- Proxies runtime YAML configs (SMS, Voice, modules) from Nacos namespaces
- Combines with MySQL entity data in the unified config endpoint

### With Redis

- Publishes CDC notifications to Redis channels (optional, per tenant)
- Used as a fast notification path complementing Kafka

## Troubleshooting

| Issue | Resolution |
|-------|-----------|
| Deploy fails with "uncommitted changes" | Commit in both config-manager submodule AND parent routesphere repo |
| CDC not starting | Verify `debezium.enabled=true` in deployed profile; check MySQL `log_bin=ON`, `binlog_format=ROW` |
| Redis notification failing | Non-critical if Kafka works; set `debezium.notify.redis.enabled=false` |
| Storm detection warning | Too many CDC events/sec; add noisy tables to `debezium.table.exclude.patterns` |
| Binds to wrong IP | Check `network.private.auto-bind=true` and that the server has a `10.10.x.x` interface |
| No reseller DBs found | Verify reseller databases follow `res_*` naming convention on the same MySQL server |
