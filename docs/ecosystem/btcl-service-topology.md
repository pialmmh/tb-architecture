# BTCL SMS Dashboard & RTC-Manager — Service Topology

## Deployment: a2psms.btcliptelephony.gov.bd:4000

### Table 1: App/Container Own IP Addresses

| App / Container | Host Server | Own IP | LXD Proxy (exposed as) | SSH Access |
|----------------|-------------|--------|----------------------|------------|
| **nginx** (reverse proxy) | sbc1 | 192.168.24.101, 114.130.145.81 | — | `$SSH/btcl/ssh sbc1` |
| **SMS-Admin** LXC (React UI + RTC-Manager) | dell-sms-master | 10.15.192.60 | 10.246.7.101:8001, 10.246.7.101:3000 | `$SSH/btcl/ssh dell-sms-master "sudo lxc exec SMS-Admin -- bash -c '<cmd>'"` |
| **dell-sms-master** host | — | 10.246.7.102, 10.246.7.101 (secondary) | — | `$SSH/btcl/ssh dell-sms-master` |
| **percona_mysql** Docker | dell-sms-master | 172.17.0.2 | host 0.0.0.0:3306 → 172.17.0.2:3306 | `$SSH/btcl/ssh dell-sms-master "sudo docker exec percona_mysql bash"` |
| **routesphere-core** | dell-sms-master | 10.10.196.x (LXC) | — | (separate LXC, not SMS-Admin) |
| **config-manager** | dell-sms-master | 10.10.196.x (LXC) | — | (separate LXC, not SMS-Admin) |

`$SSH` = `/home/mustafa/telcobright-projects/routesphere/routesphere-core/tools/ssh-automation/servers`

### Table 2: App → Target Service Connectivity

| App (source) | Target Service | Target IP:Port | User | Password | Protocol | Notes |
|-------------|---------------|----------------|------|----------|----------|-------|
| **API-Gateway** | MySQL | 10.246.7.102:3306 | `tbuser` | `Takay1takaane$` | R2DBC | MySQL sees source as `172.17.0.1` (Docker bridge) |
| **Security** | MySQL | 10.246.7.102:3306 | `tbuser` | `Takay1takaane$` | JDBC | Same Docker bridge issue |
| **FreeSwitchREST** | MySQL | 10.246.7.102:3306 | `tbuser` | `Takay1takaane$` | JDBC | DB: `btcl_sms` |
| **smsrest** | MySQL | 10.246.7.102:3306 | `tbuser` | `Takay1takaane$` | JDBC | DB: `btcl_sms` |
| **ConfigManager** | MySQL | 10.246.7.102:3306 | `tbuser` | `Takay1takaane$` | JDBC | DB: `btcl_sms` |
| **API-Gateway** | Eureka | localhost:8761 | — | — | HTTP | Service discovery |
| **API-Gateway** | Security | localhost:8080 | — | — | HTTP | JWT validation |
| **React UI** | API-Gateway | localhost:8001 (via nginx) | — | JWT Bearer | HTTP | All `/AUTHENTICATION/`, `/FREESWITCHREST/`, `/SMSREST/` |
| **nginx** | React UI | 10.246.7.101:3000 | — | — | HTTP | Static files |
| **nginx** | API-Gateway | 10.246.7.101:8001 | — | — | HTTP | API proxy |
| **nginx** | routesphere | 10.246.7.101 | — | — | HTTP | `/api/v2`, `/status/*` |
| **routesphere-core** | MySQL | (host-local) | `root` | `Takay1#$ane` | JDBC | Separate user, unaffected |
| **config-manager** | MySQL | (host-local) | `root` | `Takay1#$ane` | JDBC | Separate user, unaffected |

### Table 3: Services Inside SMS-Admin Container

| Service | Port | JAR | Spring Profile | PID (since 2025) |
|---------|------|-----|---------------|-----------------|
| Eureka-Server | 8761 | `discoveryServer-0.0.1-SNAPSHOT.jar` | — | 137789 |
| API-Gateway | 8001 | `gateway-0.0.1-SNAPSHOT.jar` | btcl | 138039 |
| ConfigManager | 7071 | `EslConfig-1.0-SNAPSHOT.jar` | btcl | 138273 |
| Security | 8080 | `security-jwt-0.0.1-SNAPSHOT.jar` | btcl | 138468 |
| FreeSwitchREST | 5071 | `FreeSwitchREST-1.0-SNAPSHOT.jar` | btcl | 138822 |
| smsrest | 6080 | `SmsRest-0.0.1-SNAPSHOT.jar` | btcl | 139047 |

### Table 4: MySQL Grants (percona_mysql on dell-sms-master)

| User | Host | Password | Used By |
|------|------|----------|---------|
| `root` | localhost | `Takay1#$ane` | routesphere, config-manager |
| `tbuser` | 172.17.0.1 | `Takay1takaane$` | RTC-Manager (all 5 services) |
| `tbuser` | 127.0.0.1 | `Takay1#$ane` | — |
| `tbuser` | localhost | `Takay1#$ane` | — |
| `tbuser` | 10.246.7.106 | `Takay1#$ane` | — |

### Nginx Routing (sbc1, port 4000 SSL → a2psms.btcliptelephony.gov.bd)

| Path | Proxied To | Service |
|------|-----------|---------|
| `/` | `http://10.246.7.101:3000` | Dashboard (serve -s build) |
| `/AUTHENTICATION/` | `http://10.246.7.101:8001` | API Gateway → Security |
| `/FREESWITCHREST/` | `http://10.246.7.101:8001` | API Gateway → FreeSwitchREST |
| `/SMSREST/` | `http://10.246.7.101:8001` | API Gateway → smsrest |
| `/api/v2` | `http://10.246.7.101` | routesphere V2 API |
| `/status/borak*/khawaja*` | `http://10.246.7.101` | Sigtran status |

### Request Flow Diagram

```
Browser → https://a2psms.btcliptelephony.gov.bd:4000/
    │
    ▼
nginx (sbc1 = 114.130.145.75:40001, port 4000 SSL)
    │
    ├── /                    → http://10.246.7.101:3000 (Dashboard static files)
    ├── /AUTHENTICATION/     → http://10.246.7.101:8001 (API Gateway)
    ├── /FREESWITCHREST/     → http://10.246.7.101:8001 (API Gateway)
    ├── /SMSREST/            → http://10.246.7.101:8001 (API Gateway)
    ├── /api/v2              → http://10.246.7.101 (routesphere V2 API)
    └── /status/*            → http://10.246.7.101 (sigtran status)

10.246.7.101 = LXD proxy on dell-sms-master → SMS-Admin container (10.15.192.60)

Inside SMS-Admin container (10.15.192.60):
    Dashboard (port 3000) — serve -s build, static React files
    API Gateway (8001) → Eureka (8761) discovers:
        ├── Security (8080)        → MySQL 10.246.7.102:3306 (tbuser)
        ├── FreeSwitchREST (5071)  → MySQL 10.246.7.102:3306 (tbuser)
        ├── smsrest (6080)         → MySQL 10.246.7.102:3306 (tbuser)
        └── ConfigManager (7071)   → MySQL 10.246.7.102:3306 (tbuser)

MySQL (10.246.7.102:3306) = Docker percona_mysql on dell-sms-master host
    Docker bridge: 172.17.0.1 → 172.17.0.2 (MySQL container)
    MySQL sees all connections from 172.17.0.1
```

### Troubleshooting Quick Commands

```bash
SSH="/home/mustafa/telcobright-projects/routesphere/routesphere-core/tools/ssh-automation/servers"

# Check all RTC-Manager ports inside SMS-Admin
$SSH/btcl/ssh dell-sms-master "sudo lxc exec SMS-Admin -- bash -c 'for p in 8761 8001 8080 5071 6080 7071 3000; do echo -n \"Port \$p: \"; lsof -i :\$p -P -n 2>/dev/null | grep LISTEN | wc -l; done'"

# Test login endpoint
$SSH/btcl/ssh dell-sms-master "sudo lxc exec SMS-Admin -- bash -c 'curl -s http://localhost:8001/AUTHENTICATION/user/authenticate -X POST -H \"Content-Type: application/json\" -d \"{\\\"email\\\":\\\"admin@btcl.gov.bd\\\",\\\"password\\\":\\\"test\\\"}\"'"

# Check Security logs for MySQL errors
$SSH/btcl/ssh dell-sms-master "sudo lxc exec SMS-Admin -- bash -c 'strings /home/RTC-Manager/Security.log | grep \"Access denied\" | tail -5'"

# Check Gateway logs for connection issues
$SSH/btcl/ssh dell-sms-master "sudo lxc exec SMS-Admin -- bash -c 'strings /home/RTC-Manager/API-Gateway.log | grep -E \"ERROR|unexpectedly\" | tail -10'"

# Test MySQL from inside SMS-Admin
$SSH/btcl/ssh dell-sms-master "sudo lxc exec SMS-Admin -- bash -c 'mysql -h 10.246.7.102 -u tbuser -pTakay1takaane\\$ btcl_sms -e \"SELECT 1\" 2>&1'"

# Restart all RTC-Manager services
$SSH/btcl/ssh dell-sms-master "sudo lxc exec SMS-Admin -- bash -c '
for port in 8761 8001 7071 8080 5071 6080; do
    PID=\$(lsof -t -i:\$port 2>/dev/null)
    [ -n \"\$PID\" ] && kill \$PID && echo \"Killed port \$port\"
done
sleep 5
cd /home/RTC-Manager/Eureka-Server && nohup java -jar target/*.jar > /home/RTC-Manager/Eureka-Server.log 2>&1 &
sleep 10
cd /home/RTC-Manager/API-Gateway && nohup java -jar target/*.jar > /home/RTC-Manager/API-Gateway.log 2>&1 &
cd /home/RTC-Manager/ConfigManager && nohup java -jar target/*.jar > /home/RTC-Manager/ConfigManager.log 2>&1 &
cd /home/RTC-Manager/Security && nohup java -jar target/*.jar > /home/RTC-Manager/Security.log 2>&1 &
sleep 5
cd /home/RTC-Manager/FreeSwitchREST && nohup java -jar target/*.jar > /home/RTC-Manager/FreeSwitchREST.log 2>&1 &
cd /home/RTC-Manager/smsrest && nohup java -jar target/*.jar > /home/RTC-Manager/smsrest.log 2>&1 &
echo \"All started\"
'"
```
