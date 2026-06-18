# dashboard_graylog

Ansible Galaxy role to deploy Graylog dashboards and field-extraction pipelines via the REST API.

## Features

- **Idempotent** -- only deploys dashboards/pipelines that don't already exist (matched by title)
- **Recreate mode** -- delete + redeploy all managed dashboards in one run (`graylog_dashboards_recreate: true`)
- **Self-contained** -- dashboard definitions are JSON files shipped with the role
- **Pipeline management** -- creates/updates Graylog pipelines that extract fields required by the dashboards
- **Graylog 7.x compatible** -- uses the `/api/views/search` + `/api/views/dashboards` + `/api/system/pipelines` APIs

---

## Dashboards

### Firewall GeoIP

VyOS netfilter logs enriched with MaxMind GeoIP data.

```
+---------------------------------------------------------------+
| Blocked Connections Timeline                          (bar)   |
+-------------------------------------------+-------------------+
| Attack Origins - World Map        (map)   | Top Source        |
|                                           | Countries  (pie)  |
+-------------------------------------------+-------------------+
| Top Source Cities                  (bar)   | Network           |
|                                           | Protocols  (pie)  |
+-------------------------------------------+-------------------+
| Top Firewall Rules                 (bar)   | Top Source IPs    |
|                                            |            (bar)  |
+--------------------------------------------+------------------+
| Top ASN                            (bar)   | Blocked by       |
|                                            | Country    (bar)  |
+--------------------------------------------+------------------+
| Recent Firewall Logs                            (messages)    |
+---------------------------------------------------------------+
```

**10 widgets** -- bar charts, pies, world map, message table

### BIND DNS Operations

BIND9 query logs with view-based split-horizon analytics and GeoIP.

```
+---------------------------------------------------------------+
| DNS Query Volume Over Time                      (bar, stack)  |
+-------------------------------------------+-------------------+
| Top Queried Domains                (bar)  | Query Types (pie) |
+-------------------------------------------+-------------------+
| Top DNS Clients                    (bar)  | DNS Response      |
|                                           | Status     (pie)  |
+-------------------------------------------+-------------------+
| Query Types Over Time        (bar, stack) | Traffic by        |
|                                           | View       (pie)  |
+-------------------------------------------+-------------------+
| External DNS Clients - World Map  (map)   | External Queries  |
|                                           | by Country (bar)  |
+-------------------------------------------+-------------------+
| External Queries by ASN           (bar)   | Refused & Failed  |
|                                           | Queries   (table) |
+-------------------------------------------+-------------------+
| Queries by View Over Time   (bar, stack)  |
+-------------------------------------------+-------------------+
| Recent DNS Queries                              (messages)    |
+---------------------------------------------------------------+
```

**13 widgets** -- bar charts, stacked timelines, pies, world map, table, message table

### Web Traffic

HTTP request logs from Traefik, HAProxy, nginx and WordPress.

```
+---------------------------------------------------------------+
| HTTP Requests Over Time                           (bar)       |
+-------------------------------------------+-------------------+
| Traefik Errors Over Time          (bar)   | Web Sources (pie) |
+-------------------------------------------+-------------------+
| HAProxy Backends                  (bar)   | Log Levels  (pie) |
+-------------------------------------------+-------------------+
| WordPress Activity                (bar)   | HTTP Status       |
|                                           | Codes      (pie)  |
+-------------------------------------------+-------------------+
| Web Error Table                 (table)   | Top URLs  (table) |
+-------------------------------------------+-------------------+
| Recent Web Logs                                 (messages)    |
+---------------------------------------------------------------+
```

**10 widgets** -- bar charts, pies, sorted tables, message table

### Proxy & AV Security

Squid multi-mode proxy (explicit + transparent IoT) with ClamAV/c-icap antivirus scanning.

```
+---------------------------------------------------------------+
| Proxy Traffic Over Time                         (bar, stack)  |
+-------------------------------------------+-------------------+
| Traffic by Port              (bar, stack) | Cache Status      |
|                                           | Distribution(pie) |
+-------------------------------------------+-------------------+
| Top Clients                       (bar)   | HTTP Response     |
|                                           | Codes      (pie)  |
+-------------------------------------------+-------------------+
| Top Domains                       (bar)   | Bump Mode         |
|                                           | Distribution(pie) |
+-------------------------------+-----------+-------------------+
| IoT Transparent               | IoT Transparent               |
| Allowed Domains        (bar)  | Blocked Domains        (bar)  |
+-------------------------------+-------------------------------+
| Explicit HTTP                 | SSL Bump                      |
| Top URLs               (bar) | Top Domains             (bar) |
+---------------------+---------+-----------+-------------------+
| ClamAV/ICAP Scan    | ICAP Response       | Denied /          |
| Activity (bar,stack) | Codes        (pie)  | Terminated  (bar) |
+---------------------+---------------------+-------------------+
| Recent Proxy Logs                                 (messages)  |
+---------------------------------------------------------------+
```

**15 widgets** -- 3 stacked timelines, 7 bar charts, 4 pies, 1 message table

Port mapping (`%>lp` reports original destination port in transparent/intercept mode):
| Port | Mode | Description |
|------|------|-------------|
| `80` | Transparent HTTP | IoT VLAN, TPROXY redirect |
| `443` | Transparent HTTPS | IoT VLAN, ssl-bump peek+splice |
| `3128` | Explicit HTTP | Server proxy, direct connect |
| `3130` | Explicit SSL Bump | Server proxy, ssl-bump |

### Infrastructure (Proxmox)

Proxmox VE hypervisor syslog events.

```
+---------------------------------------------------------------+
| Log Volume (PVE)                            (bar, stack)      |
+-------------------------------------------+-------------------+
| Proxmox Events Over Time          (bar)   | Top Sources (pie) |
+-------------------------------------------+-------------------+
| Errors (PVE)                      (bar)   | Log Levels  (pie) |
+-------------------------------------------+-------------------+
| Facilities                        (pie)   | Services    (pie) |
+-------------------------------------------+-------------------+
| Recent PVE Logs                                 (messages)    |
+---------------------------------------------------------------+
```

**8 widgets** -- bar charts, pies, message table

### K3s & Containers

Kubernetes cluster and container runtime logs.

```
+---------------------------------------------------------------+
| Log Volume (K3s)                            (bar, stack)      |
+-------------------------------------------+-------------------+
| Errors (K3s)                      (bar)   | Top Sources (pie) |
+-------------------------------------------+-------------------+
| Pods (containers)                 (pie)   | Log Levels  (pie) |
+-------------------------------------------+-------------------+
| Tags (namespaces)                 (pie)   | Images      (pie) |
+-------------------------------------------+-------------------+
| Nodes                             (pie)   |                   |
+-------------------------------------------+-------------------+
| Recent K3s Logs                                 (messages)    |
+---------------------------------------------------------------+
```

**9 widgets** -- bar charts, pies, message table

### Security & WAF

ModSecurity WAF events, CrowdSec decisions, and firewall drops.

```
+---------------------------------------------------------------+
| Security Events Over Time                         (bar)       |
+-------------------------------------------+-------------------+
| Top Firewall Rules (Drops)        (bar)   | WAF Instances     |
|                                           |            (pie)  |
+-------------------------------------------+-------------------+
| CrowdSec Decisions                (bar)   | Security          |
|                                           | Levels     (pie)  |
+-------------------------------------------+-------------------+
| Inbound Interfaces (VLANs)       (bar)   | Blocked           |
|                                           | Protocols  (pie)  |
+-------------------------------------------+-------------------+
| Security Zones Over Time    (bar, stack)  | Top Blocked       |
|                                           | Dest Ports  (bar) |
+-------------------------------------------+-------------------+
| Recent Security Events                          (messages)    |
+---------------------------------------------------------------+
```

**10 widgets** -- bar charts, stacked timeline, pies, message table

---

## Pipelines

| Pipeline | Rule(s) | Extracted Fields |
|----------|---------|-----------------|
| Firewall Log Processing | Extract Firewall Fields | `source_ip`, `destination_ip`, `network_protocol`, `fw_rule`, `destination_port`, `source_port`, `src_geolocation`, `src_country`, `src_city`, `src_as_org` |
| DNS Log Processing | Extract DNS Fields | `dns_client_ip`, `dns_domain`, `dns_view`, `dns_query_type`, `dns_status`, `dns_geolocation`, `dns_country` |
| Web Log Processing | Extract Web Fields | `http_method`, `http_url`, `http_status`, `http_response_size`, `http_client_ip` |
| Proxy Log Processing | Set Proxy Source, Extract Squid Fields | `squid_client_ip`, `squid_status`, `squid_http_code`, `squid_bytes`, `squid_method`, `squid_url`, `squid_sni`, `squid_bump_mode`, `squid_port`, `icap_mode`, `icap_response_code` |

---

## Expected Log Formats

The pipelines expect specific log formats from each source. If the format doesn't match, field extraction will silently produce empty values.

### Squid (Proxy Log Processing)

```
# squid.conf -- local log
logformat squid_multi %ts.%03tu %6tr %>a %Ss/%03>Hs %<st %rm %ru %[un %Sh/%<a %mt %ssl::>sni %ssl::bump_mode %>lp

# squid.conf -- syslog towards Graylog (RFC3164 with custom header)
logformat syslog_multi <14>%{%b %d %H:%M:%S}tl webproxy squid[0]: %ts.%03tu %6tr %>a %Ss/%03>Hs %<st %rm %ru %[un %Sh/%<a %mt %ssl::>sni %ssl::bump_mode %>lp

access_log udp://<graylog_ip>:514 syslog_multi
access_log stdio:/var/log/squid/access.log squid_multi
```

Fields in order: `EPOCH.MS  RESPONSE_TIME  CLIENT_IP  STATUS/HTTP_CODE  BYTES  METHOD  URL  USERNAME  HIERARCHY/PEER  MIME_TYPE  SNI  BUMP_MODE  LISTEN_PORT`

### c-icap / ClamAV (Set Proxy Source)

c-icap access logs forwarded via VyOS rsyslog (`source:vyos`, matched by `c-icap[` or `clamav[` in message). The pipeline renames `source` to `clamav-icap` and extracts ICAP fields.

```
# ICAP access log format (in message body):
# HOSTNAME c-icap[PID]: TIMESTAMP, SRC DST REQMOD|RESPMOD service CODE
```

### Firewall (Firewall Log Processing)

VyOS iptables/nftables log messages containing `SRC=` and `PROTO=` keywords (standard netfilter log format).

### DNS (DNS Log Processing)

BIND9 query log format with named views:
```
client @0xHEX IP#PORT (DOMAIN): view VIEWNAME: query: DOMAIN IN TYPE +flags
```

### Web (Web Log Processing)

Three formats supported (auto-detected by source name):

```
# Traefik (CLF): IP - - [date] "METHOD URL HTTP/x.x" STATUS SIZE ...
# HAProxy: ... timers STATUS SIZE ... "METHOD URL HTTP/x.x"
# nginx: ... "METHOD URL HTTP/x.x" STATUS SIZE
```

---

## Requirements

- Graylog 7.x with REST API enabled
- Ansible 2.14+
- The target host must have direct access to the Graylog API (typically `localhost:9000`)
- For GeoIP enrichment (Firewall/DNS), lookup tables must exist (managed by the graylog server role)

## Role Variables

```yaml
# Graylog API credentials
graylog_root_username: admin              # API username
vault_graylog_root_password: "changeme"   # API password (use Ansible vault)
graylog_api_url: "http://127.0.0.1:9000"  # Graylog API base URL

# Pipeline management (set false to skip)
graylog_pipelines_manage: true

# Recreate mode (delete + redeploy all dashboards)
graylog_dashboards_recreate: false

# Dashboard list (override to deploy a subset)
graylog_dashboards:
  - title: "Firewall GeoIP Dashboard"
    file: "firewall-geoip.json"
  # ...
```

## Installation

```bash
ansible-galaxy install jbsky.dashboard_graylog
```

Or in `requirements.yml`:

```yaml
roles:
  - name: jbsky.dashboard_graylog
    src: https://github.com/jbsky/dashboard-graylog
    version: main
```

## Usage

```bash
# Deploy all dashboards + pipelines
ansible-playbook site.yml --tags dashboards

# Recreate all dashboards (delete + redeploy)
ansible-playbook site.yml -e graylog_dashboards_recreate=true

# Skip pipeline management
ansible-playbook site.yml -e graylog_pipelines_manage=false
```

## Adding Custom Dashboards

1. Create a JSON file with the structure `{"dashboard": {...}, "search": {"queries": [...]}}`
2. Place it in `files/dashboards/`
3. Add an entry to `graylog_dashboards` in `defaults/main.yml`

The `state` key in the dashboard JSON must match `search.queries[0].id`.

## API Endpoints Used

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/system` | GET | Health check |
| `/api/dashboards` | GET | List existing dashboards |
| `/api/views/search` | POST | Create search definition |
| `/api/views/dashboards` | POST | Create dashboard view |
| `/api/views/{id}` | DELETE | Delete dashboard (recreate mode) |
| `/api/system/pipelines/pipeline` | GET/POST/PUT | Pipeline CRUD |
| `/api/system/pipelines/rule` | GET/POST/PUT | Pipeline rule CRUD |
| `/api/system/pipelines/connections` | GET/POST | Stream-pipeline connections |

## License

MIT
