# Sigtran SS7 Stack -- Architecture Document

## Summary

The Sigtran project is a customized fork of [Restcomm jSS7](https://github.com/Restcomm/jss7) (version 8.0.0-SNAPSHOT) that implements a full SS7 signaling stack for sending SMS via the telecom core network. It acts as the low-level SS7 gateway that Routesphere uses to send SRI (Send Routing Info) and MT-FSM (Mobile Terminated Forward Short Message) operations to mobile operator HLRs/VLRs over SCTP/M3UA/SCCP/TCAP/MAP protocols.

Each deployment runs 4 parallel JVM instances (borak1, borak2, khawaja1, khawaja2), each maintaining a separate SCTP association to different Signaling Transfer Points (STPs). This provides both load distribution and redundancy across multiple physical links.

The project is multi-tenant, supporting bdcom, link3, and btcl tenants via per-tenant JSON configuration directories.

## Tech Stack

| Component | Technology | Version |
|-----------|-----------|---------|
| Language | Java | 8 (required, uses `-Djava.ext.dirs`) |
| Base Framework | Restcomm jSS7 | 8.0.0-SNAPSHOT (forked) |
| HTTP Framework | SparkJava | 2.7.1 |
| UDP Framework | Netty NIO | 4.0.37.Final |
| Message Queue | Apache Kafka (client) | 3.3.0 |
| Redis Client | Lettuce (biz.paluch.redis) | 4.2.1.Final |
| Redis Client (alt) | Jedis | (bundled) |
| JSON | Gson | 2.8.0 |
| Rate Limiting | Guava RateLimiter | 31.0.1-jre |
| Logging | Log4j2 | 2.19.0 |
| Build | Maven | any (with Java 8) |
| Serialization | Jackson | 2.13.3 |
| License | GNU AGPL v3 | |

## Directory Structure

```
sigtran/
  pom.xml                          # Parent POM (org.restcomm.protocols.ss7:ss7-parent:8.0.0-SNAPSHOT)
  CLAUDE.md                        # Java 8 requirement noted
  logrotate.conf                   # Log rotation (3-day retention)

  # SS7 Protocol Stack Modules (upstream Restcomm)
  m3ua/                            # MTP3 User Adaptation (SCTP transport)
  sccp/                            # Signalling Connection Control Part
  tcap/                            # Transaction Capabilities Application Part
  map/                             # Mobile Application Part (SRI, MT-FSM)
  mtp/                             # Message Transfer Part
  isup/                            # ISDN User Part
  cap/                             # CAMEL Application Part
  inap/                            # Intelligent Network Application Part
  tcap-ansi/                       # TCAP ANSI variant

  # Shared Infrastructure
  scheduler/                       # Task scheduling
  congestion/                      # Congestion control
  statistics/                      # Statistics collection
  management/                      # Shell management (CLI)
  ss7-ext/                         # SS7 extensions API
  service/                         # WildFly integration (unused in prod)

  # Telcobright Custom Code
  telcobright_helper/              # Decode/helper library (MAP data classes)
    src/.../telcobright_helper/
      SigtranStackBase.java        # Interface for stack operations
      SigtranStackAdapter.java     # Adapter for stack
      SigtranStackContainer.java   # Container management
      MapDataTB.java               # MAP data structures
      InteractiveCli.java          # Interactive CLI tool
      telcobrightDecode/           # Custom ASN.1 decoders (IMSI, LMSI, SRI, MT)

  # Application (Simulator = Production Runtime)
  tools/
    simulator/
      core/src/.../telcobrightApp/
        SigtranStackTb.java           # MAIN CLASS: REST API + UDP + Kafka + Redis + SMS flow
        SigtranSmsPayload.java        # Unified payload DTO (SRI + MT requests/responses)
        SigtranUdpPayloadListener.java # Netty UDP listener (fire-and-forget)
      core/src/.../management/
        KafkaConsumer.java            # Kafka consumer (SMS tasks from Routesphere)
        KafkaProducer.java            # Kafka producer (MT-FSM responses back)
        RedisClient_Lettuce.java      # Redis client (Sentinel-aware)
        RedisClient_Jedis.java        # Redis client (alternative)
        TesterHostImpl.java           # SS7 stack host management
      bootstrap/
        src/.../bootstrap/
          Main.java                   # JVM entry point
          MainTBS.java                # Telcobright server bootstrap
          SmsServerTB.java            # SMS server implementation
          SmsClientTB.java            # SMS client implementation
          MultiStackLauncher.java     # Multi-link stack launcher
          SctpManTB.java              # SCTP management
          SccpManTB.java              # SCCP management
          TcMapManTB.java             # TCAP/MAP management
        target/
          simulator-ss7-borak1/       # Instance 1 runtime
          simulator-ss7-borak2/       # Instance 2 runtime
          simulator-ss7-khawaja1/     # Instance 3 runtime
          simulator-ss7-khawaja2/     # Instance 4 runtime

  # Operations
  start-sigtran/
    start-all-sigtran.sh            # Start all 4 instances (tenant-aware)
    stop-all-sigtran.sh             # Stop all instances
    deploy.sh                       # Systemd service + watchdog installer
    deploy-code.sh                  # Full code deployment (git pull + build + restart)
    deploy.conf                     # Watchdog/deploy configuration
    systemd-wrapper.sh              # Systemd ExecStart/ExecStop wrapper
    monitor.sh                      # Monitoring
    status-sigtran.sh               # Status checking
    view-logs.sh                    # Log viewer
    setup-logrotate.sh              # Logrotate installer

  servers/
    bdcom/keys/                     # SSH keys for bdcom server

  .idea/runConfigurations/          # IntelliJ debug configurations
```

## Key Components

### SigtranStackTb (Main Application Class)

**File:** `/home/mustafa/telcobright-projects/sigtran/tools/simulator/core/src/main/java/org/restcomm/protocols/ss7/tools/simulator/telcobrightApp/SigtranStackTb.java`

This is the central class that orchestrates everything. On startup (`startLocal()`), it:

1. Loads instance-specific JSON config via `JsonConfig`
2. Validates `tenantName` (fail-fast if missing)
3. Initializes rate limiters (default 60 TPS) and thread pool (default 10 threads)
4. Starts the SS7 stack (`host.start()`)
5. Creates Kafka consumer/producer connections
6. Starts the Spark HTTP server on `sparkAddress:sparkPort`
7. Starts the Netty UDP listener on the same `sparkAddress:sparkPort` (TCP vs UDP -- no conflict)
8. Registers REST API endpoints

### SigtranSmsPayload (Integration DTO)

**File:** `/home/mustafa/telcobright-projects/sigtran/tools/simulator/core/src/main/java/org/restcomm/protocols/ss7/tools/simulator/telcobrightApp/SigtranSmsPayload.java`

Unified payload exchanged between Routesphere and Sigtran for both SRI and MT-FSM operations. Contains:

- `payloadType`: 1 = SRI, 2 = MT
- `smsId`: Correlation key from Routesphere
- `destinationNumber`, `callingPartyNumber`, `message`, `encoding`
- `messageSequenceNumber`, `totalSegments`, `udh`: Multi-part SMS support
- `mapVersion`: Optional MAP version override (for GSM MAP version fallback)
- `sriInfo`: Nested object with `transactionId`, `imsi`, `vlr`, timestamps, error codes
- `mtFsmInfo`: Nested object with `transactionId`, timestamps, `mtRespCode`, error codes

### SigtranUdpPayloadListener (UDP Interface)

**File:** `/home/mustafa/telcobright-projects/sigtran/tools/simulator/core/src/main/java/org/restcomm/protocols/ss7/tools/simulator/telcobrightApp/SigtranUdpPayloadListener.java`

Netty-based UDP listener. Fire-and-forget pattern -- receives JSON payloads, dispatches to worker thread pool (max 50 threads), no response sent back. This is the primary interface used by Routesphere for sending SMS operations.

## External Connections

### SS7/SCTP Links (Operator Network)

Each instance maintains an SCTP association to a Signaling Transfer Point (STP):

| Instance | SCTP Local | SCTP Remote | OPC | DPC | Debug Port |
|----------|-----------|-------------|-----|-----|------------|
| **bdcom tenant** | | | | | |
| borak-1 | 10.246.3.34:4029 | 10.240.191.101:40024 | 4540 | 4702 | 8787 |
| borak-2 | 10.246.3.34:4030 | 10.240.191.102:40025 | 4540 | 4699 | 8788 |
| khawaja-1 | 10.246.3.34:4031 | 10.240.231.101:40026 | 4540 | 4700 | 8789 |
| khawaja-2 | 10.246.3.34:4032 | 10.240.231.102:40027 | 4540 | 4701 | 8790 |
| **link3 tenant** | | | | | |
| borak-1 | 10.246.23.100:3027 | 10.240.191.101:40080 | 7575 | 4702 | 8787 |
| **btcl tenant** | | | | | |
| borak-1 | 10.246.7.101:11000 | 10.240.191.101:40084 | 4688 | 4702 | 8787 |

M3UA configuration uses IPSP (IP Signalling Point) mode with SE (Single Exchange) and routing context 65535.

### Spark HTTP API (per instance)

| Instance | Spark Address (bdcom) | Port |
|----------|----------------------|------|
| borak-1 | 10.10.199.1 | 8282 |
| borak-2 | 10.10.199.1 | 8283 |
| khawaja-1 | 10.10.199.1 | 8284 |
| khawaja-2 | 10.10.199.1 | 8285 |

For link3: sparkAddress is `10.10.197.1`, port `8282`.
For btcl: sparkAddress is `10.246.7.102`, port `8282`.

### Kafka Cluster

- Brokers: `10.10.199.20:9092, 10.10.198.20:9092, 10.10.197.20:9092`
- Consumer topics: `borak-1`, `borak-2`, `khawaja-1`, `khawaja-2` (one per instance)
- Producer topic: `PLAINTEXT` (MT-FSM responses back to Routesphere)
- Each instance has its own consumer group matching its topic name

### Redis (Sentinel)

- Sentinel nodes: `10.10.199.1:26380, 10.10.198.1:26380, 10.10.197.1:26380`
- Sentinel master: `mymaster`
- Standalone fallback: `10.10.199.20:6379` (bdcom), `10.10.197.20:6379` (link3), `172.20.0.22:6379` (btcl)
- Used for: request correlation, payload storage, pub/sub response channels

## REST API Endpoints

All endpoints served by SparkJava (embedded Jetty), thread pool 400 max / 10 min / 60s idle.

| Method | Path | Description |
|--------|------|-------------|
| GET | `/hello` | Health check (returns "Hello, world") |
| GET | `/status` | Sigtran stack status (SCTP/M3UA connection state) |
| GET | `/pinginfo` | Ping information |
| POST | `/sendMessages` | Legacy Kafka-flow SMS send |
| POST | `/api/sendSri` | Send SRI request (async, fire-and-forget, returns immediately) |
| POST | `/api/sendMtfsm` | Send MT-FSM request (async, fire-and-forget, returns immediately) |

### UDP Interface

Same address and port as Spark HTTP (TCP vs UDP coexist on same port). Accepts JSON-serialized `SigtranSmsPayload` objects.

- `payloadType=1` (SRI): Dispatched to `handleUdpSriRequest()` -- calls `sendSriByHttp()`
- `payloadType=2` (MT): Dispatched to `handleUdpMtFsmRequest()` -- calls `sendMtFsmByHttp()`

This is the primary interface used by Routesphere (via `SigTranSmsRegistry` which sends UDP datagrams).

## Data Flow

### SMS Send Flow (Routesphere to SS7 Network)

```
Routesphere                          Sigtran Instance                    Operator Network
    |                                     |                                    |
    |--- UDP {payloadType=1, SRI} ------->|                                    |
    |   (fire-and-forget)                 |--- SCTP/M3UA/SCCP/TCAP/MAP ------>|
    |                                     |    SRI-for-SM request              |
    |                                     |<--- SRI response (IMSI, VLR) -----|
    |                                     |                                    |
    |                                     |  [stores payload in Redis]         |
    |                                     |  [publishes to "sriresponse-<tenant>" Redis channel]
    |                                     |                                    |
    |<-- Redis Pub/Sub (SRI response) ----|                                    |
    |                                     |                                    |
    |--- UDP {payloadType=2, MT} -------->|                                    |
    |   (with IMSI/VLR from SRI)          |--- SCTP/M3UA/SCCP/TCAP/MAP ------>|
    |                                     |    MT-ForwardSM request            |
    |                                     |<--- MT-FSM response ---------------|
    |                                     |                                    |
    |                                     |  [publishes to "mtfsmpublisher-<tenant>" Redis channel]
    |                                     |  [publishes to Kafka "PLAINTEXT" topic]
    |<-- Redis Pub/Sub (MT response) -----|                                    |
    |<-- Kafka (MT response) -------------|                                    |
```

### Request Routing (HTTP vs Kafka)

Sigtran supports two request flows, distinguished by a Redis marker:

1. **HTTP/UDP flow** (from Routesphere REST API): Sets `source_<txId> = "HTTP"` in Redis. Responses published to Redis pub/sub channels (`sriresponse-<tenant>`, `mtfsmpublisher-<tenant>`).

2. **Kafka flow** (legacy): No Redis marker. SRI response automatically triggers MT-FSM. Responses sent to Kafka producer topic.

The `onSriResponse()` and `onMtFsmResponse()` methods check the Redis `source_<txId>` key to route to the correct flow.

### Redis Key Patterns

| Key Pattern | Purpose | TTL |
|-------------|---------|-----|
| `source_<txId>` | Request source marker ("HTTP" or absent) | 300s |
| `payload_<txId>` | SRI payload JSON (for response correlation) | 300s |
| `payload_mt_<txId>` | MT-FSM payload JSON (for response correlation) | 300s |

### Redis Pub/Sub Channels

| Channel | Direction | Content |
|---------|-----------|---------|
| `sriresponse-<tenant>` | Sigtran -> Routesphere | SRI response (IMSI, VLR, error) |
| `mtfsmpublisher-<tenant>` | Sigtran -> Routesphere | MT-FSM response (success/error) |

## Configuration

### MainConfig.json (per instance)

Located at `tools/simulator/bootstrap/target/simulator-ss7-<instance>/data/MainConfig.json`. Points to the active tenant configuration directory.

```json
{
    "appDir": "bdcom",
    "sigtranInstance": "borak-1",
    "tpsLimit": "60",
    "threadPoolSize": "10"
}
```

### Instance Config JSON (per tenant per instance)

Located at `tools/simulator/bootstrap/target/simulator-ss7-<instance>/data/<tenant>/<instance-name>.json`.

Key fields:

| Field | Description | Example |
|-------|-------------|---------|
| `tenantName` | Tenant identifier | `"bdcom"` |
| `appName` | Display name | `"Borak-1"` |
| `instanceId` | Numeric ID | `"1"` |
| `localHost` / `localPort` | SCTP local bind | `"10.246.3.34"` / `"4029"` |
| `remoteHost` / `remotePort` | SCTP remote STP | `"10.240.191.101"` / `"40024"` |
| `opc` | Originating Point Code | `"4540"` |
| `dpc` | Destination Point Code | `"4702"` |
| `routingContext` | M3UA routing context | `"65535"` |
| `smscGTLocal` | Local SMSC GT | `"8809666666666"` |
| `hlrGTRemote` | Remote HLR GT | `"880711330208257"` |
| `sparkAddress` / `sparkPort` | HTTP+UDP bind | `"10.10.199.1"` / `"8282"` |
| `kafkaAddress` | Kafka brokers | `"10.10.199.20:9092,..."` |
| `consumerKafkaTopicName` | Kafka consumer topic | `"borak-1"` |
| `producerKafkaTopicName` | Kafka producer topic | `"PLAINTEXT"` |
| `redisAddress` / `redisPort` | Redis standalone | `"10.10.199.20"` / `"6379"` |
| `redisSentinelNodes` | Redis Sentinel | `"10.10.199.1:26380,..."` |
| `redisSentinelMaster` | Sentinel master name | `"mymaster"` |
| `ignoreRnCode` | Ignore routing number code | `"false"` |

### SCTP XML Config

`SimSCTPServer_<instance>_sctp.xml` -- defines SCTP client association:

```xml
<associations>
    <name value="Ass_borak-1"/>
    <association name="Ass_borak-1" assoctype="CLIENT"
        hostAddress="10.246.3.34" hostPort="4029"
        peerAddress="10.240.191.101" peerPort="40024"
        ipChannelType="0" extraHostAddresseSize="0"/>
</associations>
```

### M3UA XML Config

`SimM3uaServer_<instance>_m3ua1.xml` -- defines ASP factory, AS, and routing:

```xml
<aspFactoryList>
    <aspFactory name="testasp" assocName="Ass_borak-1" started="true"
        maxseqnumber="256" aspid="3" heartbeat="false"/>
</aspFactoryList>
<asList>
    <as name="testas" functionality="IPSP" exchangeType="SE" ipspType="CLIENT">
        <routingContext><rc value="65535"/></routingContext>
        <trafficMode mode="2"/>
    </as>
</asList>
<route>
    <key value="4702:4540:3"/>
    <routeAs trafficModeType="2" as="testas"/>
</route>
```

### JVM Configuration (per instance run.sh)

- Heap: `-Xms1g -Xmx2g`
- IPv4 forced: `-Djava.net.preferIPv4Stack=true`
- Headless mode: `-Djava.awt.headless=true`
- Debug ports: borak1=8787, borak2=8788, khawaja1=8789, khawaja2=8790
- Entry class: `org.restcomm.protocols.ss7.tools.simulator.bootstrap.Main`

### deploy.conf (Watchdog/Operations)

```
PROJECT_ROOT=/home/bdcom/sigtran/sigtran
CHECK_INTERVAL=30        # Watchdog health check interval (seconds)
RESTART_DELAY=10         # Delay before restart (seconds)
MAX_RESTART_ATTEMPTS=5   # Max restarts per hour before giving up
AUTOSTART_ENABLED=true   # Systemd autostart
WATCHDOG_ENABLED=true    # Process watchdog
LOG_ROTATION_DAYS=3
LOG_MAX_SIZE=100M
```

## Deployment

### Build

```bash
# Requires Java 8
export JAVA_HOME=/path/to/java8
mvn clean install -pl tools/simulator/core,tools/simulator/bootstrap -am -DskipTests
```

The build produces 4 instance directories under `tools/simulator/bootstrap/target/`:
- `simulator-ss7-borak1/`
- `simulator-ss7-borak2/`
- `simulator-ss7-khawaja1/`
- `simulator-ss7-khawaja2/`

Each contains `bin/run.sh`, `lib/`, `conf/`, and `data/<tenant>/` directories.

### Deployment Scripts

**Full deploy** (`deploy-code.sh`):
```bash
./start-sigtran/deploy-code.sh deploy bdcom    # git pull + Java 8 build + config fix + restart
./start-sigtran/deploy-code.sh rollback        # Rollback 1 step
./start-sigtran/deploy-code.sh version         # Show deployed version
./start-sigtran/deploy-code.sh history         # Deploy history (JSONL-based)
```

Features: pre-flight checks, git tagging (`deploy/<tenant>/<seq>`), JSONL deploy history, rollback support.

**Service management** (`deploy.sh`):
```bash
sudo ./start-sigtran/deploy.sh install bdcom   # Install systemd service + auto-enable
sudo systemctl start sigtran                   # Start service
sudo systemctl status sigtran                  # Check status
```

Creates a systemd service unit at `/etc/systemd/system/sigtran.service` (Type=oneshot, RemainAfterExit=yes).

**Manual start/stop**:
```bash
./start-sigtran/start-all-sigtran.sh bdcom     # Start all 4 instances for bdcom
./start-sigtran/start-all-sigtran.sh stop      # Stop all
./start-sigtran/start-all-sigtran.sh status    # Status
```

### Production Server Layout

On the production server (e.g., bdcom):
```
/home/bdcom/sigtran/sigtran/              # PROJECT_ROOT
    tools/simulator/bootstrap/target/     # Instance runtime dirs
    logs/                                 # Console logs (borak1.log, etc.)
    start-sigtran/                        # Operations scripts
    deploy-history.jsonl                  # Deploy history
```

### Watchdog

The systemd wrapper (`systemd-wrapper.sh`) starts all 4 instances plus a watchdog process that:
- Checks every 30 seconds if each instance process is alive
- Automatically restarts crashed instances (up to 5 attempts per hour)
- Logs to `logs/watchdog.log`

### Log Rotation

`logrotate.conf` configures:
- Console logs (`logs/*.log`): daily rotation, 3-day retention, compressed
- Server logs (`target/simulator-ss7-*/log/server.log`): daily, 3 days
- Temp logs (`/tmp/borak*.log`): daily, 1 day

### Remote Debug

IntelliJ run configurations pre-configured for remote debugging:
- borak1: `10.10.199.1:8787`
- borak2: `10.10.199.1:8788`
- borak3 (khawaja1): `10.10.199.1:8789`
- borak4 (khawaja2): `10.10.199.1:8790`
- link3 variants also available (same ports, different host)

## Integration Points with Routesphere

### Routesphere -> Sigtran (Outbound SMS)

Routesphere sends SMS operations to Sigtran via **UDP datagrams** (primary) or **HTTP POST** (alternative). The `SigTranSmsRegistry` in Routesphere selects a Sigtran instance and sends a JSON-serialized `SigtranSmsPayload` to the instance's `sparkAddress:sparkPort` over UDP.

- **SRI request**: `payloadType=1`, contains `smsId`, `destinationNumber`
- **MT-FSM request**: `payloadType=2`, contains full SMS data plus IMSI/VLR from prior SRI response

### Sigtran -> Routesphere (Responses)

Responses flow back via two channels:

1. **Redis Pub/Sub** (primary for HTTP/UDP flow):
   - Channel `sriresponse-<tenant>`: SRI response with IMSI, VLR, error codes
   - Channel `mtfsmpublisher-<tenant>`: MT-FSM response with delivery status

2. **Kafka** (for MT-FSM responses):
   - Topic `PLAINTEXT`: Full `MessageDetails` JSON with delivery status

### Shared Data Structures

Routesphere and Sigtran share the `SigtranSmsPayload` class structure (not as a library dependency, but as a JSON contract). Key contract fields:

- `smsId` -- unique correlation key
- `payloadType` -- 1=SRI, 2=MT
- `sriInfo.imsi`, `sriInfo.vlr` -- HLR lookup results
- `mtFsmInfo.mtRespCode` -- "200" = success
- `mapVersion` -- optional MAP version override for fallback (v3 default, v2/v1 explicit)

### Tenant-Specific Redis Channels

The channel names are suffixed with the tenant name from the instance config:
- `sriresponse-bdcom`, `sriresponse-link3`, `sriresponse-btcl`
- `mtfsmpublisher-bdcom`, `mtfsmpublisher-link3`, `mtfsmpublisher-btcl`

This ensures multi-tenant Routesphere instances only receive responses for their tenant.

## Multi-Part SMS Handling

For concatenated SMS (messages longer than 160 chars / 70 UCS-2 chars):

1. Routesphere splits the message into segments and sends each as a separate MT-FSM via UDP
2. Each segment carries `messageSequenceNumber`, `totalSegments`, `udh` (User Data Header), and `referenceNumber`
3. In the legacy Kafka flow, Sigtran handles tail messages sequentially with a 100ms delay between segments to avoid "subscriberBusyForMT-SMS" errors
4. In the HTTP/UDP flow, Routesphere manages segment sequencing externally

## Rate Limiting

Each Sigtran instance applies rate limiting via Guava `RateLimiter`:
- Default: 60 TPS (configurable via `tpsLimit` in MainConfig.json)
- Separate 10 TPS limiter for certain operations
- Bounded thread pools: 50 UDP workers, 100 async REST threads, configurable SMS processor pool (default 10)
