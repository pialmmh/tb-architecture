# Night Watcher

## Summary

Night Watcher is Telcobright's unified operations platform for the Routesphere ecosystem. It bundles security monitoring (WAF, SIEM, intrusion prevention), high-availability cluster management, and identity management into a single deployable container per tenant. The system runs as an all-in-one Docker or LXC container managed by supervisord, with 10+ supervised processes providing defense-in-depth security, backend health monitoring with automatic nginx failover, quorum-based HA with VIP failover for sigtran/SS7 services, and a React-based security dashboard with Keycloak JWT authentication.

Repository: `/home/mustafa/telcobright-projects/routesphere/night-watcher/`

## Tech Stack

| Layer | Technology | Version/Detail |
|-------|-----------|----------------|
| HA Controller | Go | 1.21, compiled to single `hactl` binary |
| Security Dashboard | React 18, Vite 5, MUI 5 | SPA with Recharts, React Router 6, dayjs |
| Log Shipper | Python 3 | mysql-connector-python for MySQL inserts |
| Module Status API | Python 3 | stdlib `http.server`, no external deps |
| Watchdog | Bash | curl-based health checks, nginx upstream swap |
| Reverse Proxy / WAF | Nginx 1.22 + ModSecurity 3 (OWASP CRS v4) | Dynamic module |
| Intrusion Prevention | CrowdSec (iptables bouncer), Fail2Ban | Community blocklists |
| SIEM | Wazuh 4.x (Manager + Indexer/OpenSearch + Dashboard) | File integrity, rootcheck, vulnerability detection |
| Cluster Coordination | HashiCorp Consul | Raft consensus, leader election, KV store |
| Identity | Keycloak | Realm `night-watcher`, client `nw-dashboard`, roles: admin/operator/viewer |
| Process Manager | supervisord | 11 managed processes |
| Container Runtime | Docker (host network) or LXD/LXC (Debian 12) | 2 GB RAM budget |
| Database | MySQL (external) | 5 partitioned tables in `security_monitoring` database |
| Base OS | Debian 12 (slim) | Both build and runtime |

### Go Dependencies (ha-controller)

- `github.com/hashicorp/consul/api` v1.29.1 -- Consul client
- `golang.org/x/crypto` v0.28.0 -- SSH executor
- `gopkg.in/yaml.v3` -- YAML config parsing

### Dashboard Dependencies (npm)

- `@mui/material` 5.15, `@emotion/react`, `@emotion/styled` -- UI framework
- `react` 18.2, `react-dom`, `react-router-dom` 6.22 -- SPA routing
- `recharts` 2.12 -- Charts
- `dayjs` 1.11 -- Date handling

## Directory Structure

```
night-watcher/
|-- Dockerfile                    3-stage build: modsec + go + runtime
|-- build.sh                      Docker image builder
|-- deploy.sh                     Single-tenant Docker deploy
|-- deploy-bdcom.sh               Multi-node LXC cluster deploy (3 nodes)
|-- launch.sh                     LXC container launcher from config file
|-- supervisord.conf              11 process definitions with priority ordering
|-- entrypoint.sh                 Container init: reads tenant.conf, wires all components
|
|-- ha-controller/                Go HA control plane
|   |-- cmd/hactl/main.go         CLI entry point + API server on :7102
|   |-- internal/
|   |   |-- api/                  HTTP status API (/status, /failover)
|   |   |-- config/               YAML config loader + validation
|   |   |-- consul/               Consul client, leader election, KV CAS
|   |   |-- engine/               Sentinel-aware reconciliation loop
|   |   |-- executor/             Local + SSH command execution
|   |   |-- healthcheck/          Ping, TCP, HTTP, Script probes
|   |   |-- model/                Domain model: cluster, node, state, policy, tenant
|   |   |-- node/                 Node abstraction + fencing
|   |   |-- resource/             Resource types: VIP, Action, Noop
|   |   |-- sentinel/             SDOWN/ODOWN consensus + failover coordinator
|   |-- configs/ha-controller.yml Sample config
|   |-- go.mod
|   |-- Makefile
|
|-- dashboard/                    React 18 + Vite + MUI security dashboard
|   |-- src/
|   |   |-- auth/                 AuthContext, ProtectedRoute, keycloak.js
|   |   |-- pages/                Login, Profile, UserManagement, Overview, HaCluster,
|   |   |                         SecurityEvents, Waf, Watchdog, Modules, Logs, Network,
|   |   |                         GatewayOverview, GatewayAudit, GatewayPolicies, KeycloakAdmin
|   |   |-- components/           Layout, AlertTable, AlertTrend, SeverityCards, TopIPs,
|   |   |                         TopRules, WafEvents, WatchdogStatus, MitreTags, DateRangePicker
|   |   |-- api/                  opensearch.js, gateway.js
|   |-- package.json
|
|-- common/                       Shared configs across all tenants
|   |-- scripts/
|   |   |-- watchdog.sh           Backend health monitoring + nginx upstream failover
|   |   |-- log-shipper.py        Tails alert logs, inserts into MySQL
|   |   |-- module-status-api.py  HTTP JSON API on :7101 (aggregates all module statuses)
|   |   |-- install-agent.sh      Remote Wazuh agent installer
|   |   |-- wazuh-manager-wrapper.sh  Foreground wrapper for supervisord
|   |-- mysql/schema.sql          5 tables, all monthly-partitioned
|   |-- nginx/snippets/           proxy_headers, modsecurity, cache, acl
|   |-- modsecurity/              Base WAF config + CRS include chain
|   |-- wazuh/ossec-base.conf     Base SIEM config (log collection, FIM, rootcheck, vuln-detect)
|
|-- tenants/
|   |-- btclsms/                  BTCL tenant
|   |   |-- tenant.conf           SSH, MySQL, Wazuh, HA settings
|   |   |-- ha-controller.yml     Cluster config (nodes, resources, health checks)
|   |   |-- nodes.json            Node list for dashboard
|   |   |-- nginx/                Site configs, SSL params
|   |   |-- modsecurity/          CRS exclusions
|   |   |-- crowdsec/             Acquisition, whitelist, collections
|   |   |-- fail2ban/             Jail config, custom filters
|   |   |-- wazuh/                Tenant-specific ossec-tenant.conf
|   |   |-- watchdog/services.conf  Backend services to health-check
|   |-- bdcom/                    BDCOM tenant (same structure + launch-*.conf for LXC)
|
|-- docs/                         Design documents (identity, 2FA, HA, API gateway, etc.)
```

## Key Components

### 1. HA Controller (`hactl`)

A Go binary implementing quorum-based health consensus modeled after Redis Sentinel.

**Reconciliation loop (5-second tick):**
1. Each node runs health checks and publishes its observation to Consul KV at `ha-controller/{cluster}/observations/{node}`
2. **SDOWN** (subjective down): a node marks the active node as down after N consecutive check failures (`fail_threshold`, default 3)
3. **ODOWN** (objective down): if `quorum` nodes agree on SDOWN, the active node is objectively down
4. **Failover**: the Consul lock holder (coordinator) selects the best candidate and executes the failover sequence

**Failover sequence on promoting node-B over node-A:**
1. CAS-increment the failover epoch in Consul KV (prevents double failover)
2. Deactivate resources on A in reverse order (notify API, stop service, remove VIP)
3. Activate resources on B in forward order (assign VIP, start service, notify API)
4. Update active node in Consul KV

**Health check types:** ping, TCP connect, HTTP (with expected body match), script (shell command with expected output match). Each check has a configurable scope: `cluster` (all nodes check, used for SDOWN/ODOWN) or `self` (local readiness, used for candidate selection).

**Resource types:**
- `vip` -- floating IP via `ip addr add/del` + gratuitous ARP
- `action` -- generic shell commands with activate/deactivate/check
- `noop` -- dummy for testing

**API endpoints (port 7102):**
- `GET /status` -- full cluster state (activeNode, sdown, odown, observations, groups)
- `POST /failover` -- manual failover with `{"targetNode": "node2"}` (coordinator only)

**Anti-flap:** `max_failovers` (default 3) within `failover_window` (default 1h). Auto-failback is configurable (`auto_failback`, default false).

### 2. Watchdog

Bash script (`common/scripts/watchdog.sh`) that monitors backend HTTP health endpoints defined in per-tenant `watchdog/services.conf`.

**services.conf format:** `name|health_url|primary_addr|standby_addr|interval_sec|drain_wait_sec`

**Behavior:**
- Polls each service's health URL via curl every `MIN_INTERVAL` seconds (default 5)
- After 3 consecutive failures: initiates drain period, then swaps nginx upstream from primary to standby
- On primary recovery while on standby: automatic failback
- Logs all events (failover, failback, drain, both_down) to MySQL `watchdog_events` table and local log file

**BTCL example monitors:** main-app (:3001), admin-app (:3000), ofbiz (:8080), report (:8083), nextjs (:30000).

### 3. Log Shipper

Python daemon (`common/scripts/log-shipper.py`) that tails three log files and inserts structured records into MySQL:

| Source Log | Target Table | Data Captured |
|-----------|-------------|---------------|
| `/var/ossec/logs/alerts/alerts.json` | `wazuh_alerts` | rule_id, level, description, agent, src_ip, groups, raw JSON |
| `/var/log/fail2ban.log` | `ban_events` | jail, action (ban/unban), IP |
| `/var/log/crowdsec_decisions.log` | `ban_events` | scenario, action type, IP, duration |

Handles file rotation (inode change detection), batches inserts (batch size 50), auto-reconnects to MySQL on failure (30s delay).

### 4. Module Status API

Python HTTP server (`common/scripts/module-status-api.py`) on port 7101, aggregating real-time status from all security modules into a single JSON response.

**Endpoints:**
- `GET /` or `GET /status` -- all modules combined
- `GET /fail2ban` -- Fail2Ban jail details (failed, banned, total_banned, banned_ips)
- `GET /crowdsec` -- decisions, alerts, bouncers, collections
- `GET /hactl` -- proxied from `http://127.0.0.1:7102/status`

Queries supervisord, fail2ban-client, cscli, wazuh PID files, nginx log line counts.

### 5. Security Dashboard

React 18 SPA with MUI, served by Nginx, authenticated via Keycloak JWT.

**Pages:** Overview, SecurityEvents, Waf, Watchdog, Modules, Logs, Network, HaCluster, GatewayOverview, GatewayAudit, GatewayPolicies, Login, Profile, UserManagement, KeycloakAdmin.

**Auth flow:** Login form posts credentials to Keycloak, receives JWT stored in sessionStorage. All API requests include `Authorization: Bearer <token>`. Auto-refresh before expiry, 401 redirects to `/login`.

**Dev port:** 7100 (Vite). Production: served as static files via Nginx.

### 6. Security Bundle (Nginx + WAF + SIEM)

**Nginx + ModSecurity 3:** Reverse proxy with OWASP Core Rule Set v4. Default mode: DetectionOnly (log, do not block). Can be switched to enforcement per tenant.

**CrowdSec:** Community IP reputation + rate limiting via iptables firewall bouncer. Per-tenant acquisition config, whitelist, and collections.

**Fail2Ban:** Log-based IP banning from ModSecurity/nginx logs. Per-tenant jail config and custom filters.

**Wazuh Manager + Indexer + Dashboard:**
- Log collection from nginx, modsecurity, fail2ban, crowdsec, auth.log, syslog
- File integrity monitoring (FIM) on `/etc/nginx`, `/etc/fail2ban`, `/etc/crowdsec`, `/etc/ssh/sshd_config`, passwd/shadow/group
- Rootcheck (trojans, devices, system, PIDs, ports, interfaces)
- Vulnerability detection (Debian bookworm provider, 12h interval)
- Agent registration on port 1515, remote communication on port 1514/tcp
- Indexer (OpenSearch) with 384MB heap, Dashboard with 256MB Node heap

## External Connections

| Target | Protocol | Port | Direction | Purpose |
|--------|----------|------|-----------|---------|
| MySQL (`security_monitoring` DB) | TCP | 3306 | Outbound | Alert storage, watchdog events, ban events, nginx stats, HA events |
| Consul cluster | TCP | 8500 | Bidirectional | Leader election, KV store for observations/epochs/active node |
| Routesphere core | HTTP | 19999 | Outbound | Health check (`/q/health`), HA notification (`/api/ha/promoted`, `/api/ha/demoted`) |
| Sigtran | HTTP | 8284 | Outbound | Health check (`/pinginfo`, expect `sctpUp`) |
| Redis Sentinel | TCP | 26380 | Outbound | Health check (TCP connect) |
| Kafka broker | TCP | 9092 | Outbound | Health check (TCP connect) |
| Keycloak | HTTP | 7104 | Internal (proxied via Nginx at `/auth/`) | JWT issuance, user management |
| Backend services (BTCL) | HTTP | 3000-8083 | Outbound | Watchdog health checks |
| Wazuh agents on remote servers | TCP | 1514, 1515 | Inbound | Agent communication and registration |
| CrowdSec Central API | HTTPS | 443 | Outbound | Community blocklist updates |
| Peer nodes (SSH) | TCP | 22 | Outbound | Remote command execution for failover, fencing |

## Data Flow

### Security Monitoring Flow

```
HTTP traffic --> CrowdSec (IP reputation) --> Nginx + ModSecurity (WAF) --> Fail2Ban (repeat offenders)
                                                          |
                                    +---------------------+---------------------+
                                    |                     |                     |
                              access.log          modsec_audit.log        error.log
                                    |                     |                     |
                                    +--------- Wazuh Manager --------+
                                                     |
                                              alerts.json
                                                     |
                                    +--------- Log Shipper --------+
                                    |                               |
                             wazuh_alerts (MySQL)           ban_events (MySQL)
                                    |
                         Wazuh Indexer (OpenSearch)
                                    |
                         Wazuh Dashboard (:5601)
                         Security Dashboard (:7100/nginx)
```

### HA Failover Flow

```
All nodes (5s tick):
  run health checks --> publish observation to Consul KV

Each node evaluates:
  consecutive failures >= fail_threshold? --> mark SDOWN

Coordinator (Consul lock holder) evaluates:
  SDOWN count >= quorum? --> ODOWN --> begin failover

Failover:
  CAS epoch increment --> deactivate on old node (reverse) --> activate on new node (forward) --> update Consul KV
```

### Watchdog Health Check Flow

```
watchdog.sh (every 5s) --> curl backend health URLs
  |
  +-- healthy: if on standby and primary recovered --> failback (swap nginx upstream)
  |
  +-- unhealthy (3 consecutive): drain period --> check standby --> swap nginx upstream
  |
  +-- log to MySQL watchdog_events + local log
```

## Configuration

### Tenant Configuration (`tenant.conf`)

Bash-sourceable key-value file. Read by `entrypoint.sh` and `deploy.sh`.

```bash
tenant_name=bdcom
ssh_tenant=bdcom           # SSH automation inventory name
ssh_server=bdcom1          # Target server name
mysql_host=10.10.199.10
mysql_port=3306
mysql_user=security_bundle
mysql_pass=changeme
mysql_db=security_monitoring
hactl_enabled=true
hactl_node_id=bdcom-nw-1
container_memory=2g
container_ports=80,443,1514,1515,5601,9000
```

### HA Controller Configuration (`ha-controller.yml`)

YAML file defining cluster, consul, nodes, resource groups, and health checks. See `/home/mustafa/telcobright-projects/routesphere/night-watcher/ha-controller/configs/ha-controller.yml` for the full reference.

Key sections:
- `cluster`: name, tenant, quorum, fail_threshold, check_interval, auto_failback, max_failovers, failover_window, observation_stale
- `consul`: address, datacenter, session_ttl, lock_delay
- `nodes[]`: id, address, priority, ssh (user, key_path, port), fence_cmd
- `groups[]`: id, resources[] (type: vip/action/noop, attrs), checks[] (type: ping/tcp/http/script, target, expect, scope: cluster/self)

### LXC Launch Configuration (`launch-*.conf`)

Bash-sourceable file for `launch.sh`. Defines container name, base image, network (bridge, IP, gateway, DNS), resource limits (memory, CPU), tenant identity, HA settings, and MySQL connection.

### Watchdog Services Configuration (`services.conf`)

Pipe-delimited file: `name|health_url|primary_addr|standby_addr|interval_sec|drain_wait_sec`

### Per-Tenant Overrides

Each tenant directory can contain subdirectories for: `nginx/` (site configs, snippets, SSL), `modsecurity/` (CRS exclusions), `crowdsec/` (acquisition, whitelist, collections), `fail2ban/` (jail.local, custom filters), `wazuh/` (ossec-tenant.conf), `watchdog/` (services.conf).

## MySQL Schema

Database: `security_monitoring` (per-tenant MySQL instance)

| Table | Purpose | Partition Key |
|-------|---------|---------------|
| `wazuh_alerts` | All Wazuh SIEM alerts (rule_id, level, src_ip, raw JSON) | `alert_timestamp` (monthly) |
| `watchdog_events` | Backend health events (failover, failback, drain, both_down) | `created_at` (monthly) |
| `ban_events` | Fail2Ban + CrowdSec ban/unban events | `event_timestamp` (monthly) |
| `nginx_hourly_stats` | Aggregated nginx traffic (2xx/3xx/4xx/5xx, WAF blocks, unique IPs) | `hour_start` (monthly) |
| `ha_events` | HA controller events (promoted, demoted, failover, fence, VIP changes) | `event_timestamp` (monthly) |

All tables use monthly partitions (p2026_01 through p2026_12 + p_future). Schema: `/home/mustafa/telcobright-projects/routesphere/night-watcher/common/mysql/schema.sql`

## Ports

| Port | Service | Binding |
|------|---------|---------|
| 80/443 | Nginx (HTTP/HTTPS reverse proxy + WAF) | Host network |
| 1514 | Wazuh agent communication (TCP) | Host network |
| 1515 | Wazuh agent registration | Host network |
| 5601 | Wazuh Dashboard (OpenSearch Dashboards) | Host network |
| 7100 | Security Dashboard (Vite dev / Nginx static) | Host network |
| 7101 | Module Status API (Python HTTP) | 127.0.0.1 only |
| 7102 | hactl status API (Go HTTP) | Host network |
| 7104 | Keycloak (proxied via Nginx at `/auth/`) | Internal |
| 9200 | Wazuh Indexer (OpenSearch) | Host network |
| 55000 | Wazuh API | Host network |

## Supervised Processes

Started by supervisord in priority order:

| Priority | Process | Command | Autostart |
|----------|---------|---------|-----------|
| 10 | nginx | `/usr/sbin/nginx -g "daemon off;"` | always |
| 12 | consul | `consul agent -config-dir=/etc/consul.d` | always |
| 15 | hactl | `/usr/local/bin/hactl --config ... --node ...` | `HACTL_ENABLED` env (default: false) |
| 20 | crowdsec | `/usr/bin/crowdsec -c /etc/crowdsec/config.yaml` | always |
| 30 | fail2ban | `fail2ban-server -f -x -v start` | always |
| 35 | wazuh-indexer | OpenSearch with 384MB heap | always |
| 40 | wazuh-manager | via wrapper script monitoring analysisd PID | always |
| 50 | wazuh-dashboard | OpenSearch Dashboards with 256MB Node heap | always |
| 55 | module-status-api | Python HTTP on :7101 | always |
| 60 | watchdog | `watchdog.sh` health check loop | always |
| 70 | log-shipper | Python tail + MySQL insert loop | always |

## Deployment

### Docker Deploy (Single Tenant)

Script: `deploy.sh <tenant>`

Steps:
1. Pre-flight: SSH access, Docker version, vm.max_map_count (262144)
2. Create MySQL tables from `common/mysql/schema.sql`
3. Tar + SCP tenant config to `/opt/security-bundle/config/` on remote
4. Save + transfer Docker image, or pull on remote
5. Stop/remove existing `security-bundle` container
6. `docker run -d --network host --memory 2g` with volume mounts for config (read-only), logs, SSL certs, Wazuh data, OpenSearch indices
7. Verify: container status, supervisorctl status

### LXC Cluster Deploy (Multi-Node)

Script: `deploy-bdcom.sh [step]`

Steps (run individually or all):
1. **image**: SCP LXC image tarball to all 3 servers, import with alias
2. **launch**: Run `launch.sh` with per-node config on each server
3. **consul**: Configure 3-node Consul cluster (`bootstrap_expect=3`, `retry_join`)
4. **mysql**: Create `security_monitoring` database and tables
5. **configs**: Push ha-controller.yml, nodes.json, nginx configs into containers
6. **verify**: Check Consul members, supervisord status, hactl /status, dashboard

### LXC Launch (`launch.sh`)

Launches a single LXC container from a config file. Configures static IP via systemd-networkd, sets resource limits (memory, CPU), enables nesting, passes environment variables (HACTL, CONSUL, MYSQL), creates `/config/tenant.conf` inside container, runs `entrypoint.sh`.

### Container Entrypoint (`entrypoint.sh`)

Executed on container start. Reads `tenant.conf`, then:
1. Copies tenant-specific nginx, modsecurity, crowdsec, fail2ban, wazuh configs
2. Merges base + tenant ossec.conf for Wazuh
3. Generates self-signed TLS certificates for Wazuh Indexer and Dashboard
4. Writes `/etc/security-bundle.env` with MySQL and HA settings for child processes
5. Validates nginx config
6. Starts supervisord in background
7. Waits for OpenSearch to start, runs `indexer-security-init.sh`
8. Waits for supervisord (foreground)

## Integration Points

### With Routesphere Core

- **Health check target**: hactl checks `http://<routesphere>:19999/q/health` (expects "UP")
- **HA notification**: on failover, hactl POSTs to `/api/ha/promoted` and `/api/ha/demoted` on routesphere
- **Watchdog**: monitors routesphere and related backend services via HTTP health URLs
- **Nginx reverse proxy**: routes external HTTPS traffic to routesphere and other backend apps

### With Sigtran

- **Health check**: hactl checks `http://<vip>:8284/pinginfo` (expects `sctpUp`)
- **Service management**: hactl starts/stops sigtran via `lxc exec` action commands
- **VIP failover**: floating IP moves between nodes so remote SS7 peers see a stable address

### With Consul

- Leader election for coordinator role (which node orchestrates failovers)
- KV store for: active node, failover epoch, per-node health observations
- Session TTL: 15s, Lock delay: 5s

### With Keycloak

- Realm: `night-watcher`, Client: `nw-dashboard` (public SPA, direct access grants)
- Roles: `admin`, `operator`, `viewer`
- Proxied via Nginx at `/auth/` to port 7104
- Dashboard uses JWT for all API calls

### With MySQL

- External MySQL instance per tenant (not inside the container)
- Database: `security_monitoring`
- Written to by: log-shipper (wazuh_alerts, ban_events), watchdog (watchdog_events)
- 5 monthly-partitioned tables

## Current Tenants

| Tenant | Deploy Method | Nodes | HA Enabled | MySQL Host |
|--------|--------------|-------|------------|------------|
| btclsms | Docker (deploy.sh) | dell-sms-master | yes | 10.10.196.10 |
| bdcom | LXC cluster (deploy-bdcom.sh) | bdcom1 (10.10.199.200), bdcom2 (10.10.198.200), bdcom3 (10.10.197.200) | yes, 3-node Consul | 10.10.199.10 |

## Alert Channels

Night Watcher does **not** currently implement outbound alerting (email, Slack, webhook). All monitoring data is:
- Stored in MySQL tables (queryable)
- Visible in the Wazuh Dashboard (port 5601)
- Visible in the Security Dashboard (port 7100)
- Available via the Module Status API (port 7101, JSON)
- Logged to local files under `/var/log/security-bundle/`

Wazuh's email alerting is configured but disabled in the base config (`<email_notification>no</email_notification>`). It could be enabled per-tenant via `ossec-tenant.conf`.
