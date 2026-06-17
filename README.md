# dashboard_graylog

Ansible role to deploy and manage Graylog dashboards and their supporting field-extraction pipelines via the REST API.

## Features

- **Idempotent**: only deploys dashboards/pipelines that don't already exist (matched by title)
- **Recreate mode**: delete + redeploy all managed dashboards in one run
- **Self-contained**: dashboard definitions are JSON files shipped with the role
- **Pipeline management**: creates/updates Graylog pipelines that extract fields required by the dashboards
- **Graylog 7.x compatible**: uses the `/api/views/search` + `/api/views/dashboards` + `/api/system/pipelines` APIs

## Included Dashboards

| Dashboard | Description |
|-----------|-------------|
| Firewall GeoIP | VyOS firewall logs with source/destination IP, GeoIP, protocols |
| BIND DNS Operations | DNS queries, clients, domains, response status, GeoIP |
| Web Traffic | HTTP requests from Traefik/HAProxy/nginx with status codes, top URLs |
| Infrastructure (Proxmox) | PVE hypervisor events, facilities, error tracking |
| K3s & Containers | Kubernetes pods, namespaces, images, nodes |
| Security & WAF | WAF events, blocked requests, attack patterns |
| Proxy & AV Security | Squid proxy logs, ICAP antivirus scanning |

## Included Pipelines

| Pipeline | Rule(s) | Extracted Fields |
|----------|---------|-----------------|
| Firewall Log Processing | Extract Firewall Fields | `source_ip`, `destination_ip`, `network_protocol`, `fw_rule`, `destination_port`, `source_port`, `src_geolocation`, `src_country`, `src_city`, `src_as_org` |
| DNS Log Processing | Extract DNS Fields | `dns_client_ip`, `dns_domain`, `dns_view`, `dns_query_type`, `dns_status`, `dns_geolocation`, `dns_country` |
| Web Log Processing | Extract Web Fields | `http_method`, `http_url`, `http_status`, `http_response_size`, `http_client_ip` |
| Proxy Log Processing | Set Proxy Source, Extract Squid Fields | `squid_client_ip`, `squid_status`, `squid_http_code`, `squid_sni`, `squid_bump_mode`, `icap_mode`, `icap_response_code` |

## Requirements

- Graylog 7.x with REST API enabled
- Ansible 2.14+
- The target host must have direct access to the Graylog API (typically `localhost:9000`)
- For GeoIP enrichment (Firewall/DNS pipelines), lookup tables must exist (managed by the graylog server role)

## Role Variables

```yaml
# Graylog API credentials
graylog_root_username: admin              # API username
vault_graylog_root_password: "changeme"   # API password (use Ansible vault)
graylog_api_url: "http://127.0.0.1:9000"  # Graylog API base URL

# Pipeline management (set false to skip pipeline deployment)
graylog_pipelines_manage: true

# Recreate mode (delete + redeploy dashboards)
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

### Basic playbook

```yaml
- hosts: graylog.example.com
  roles:
    - role: jbsky.dashboard_graylog
      tags: [dashboards]
```

### Recreate all dashboards

```yaml
- hosts: graylog.example.com
  roles:
    - role: jbsky.dashboard_graylog
      vars:
        graylog_dashboards_recreate: true
      tags: [dashboards]
```

Or via extra vars:

```bash
ansible-playbook site.yml -e graylog_dashboards_recreate=true
```

### Deploy only specific dashboards

```yaml
- hosts: graylog.example.com
  roles:
    - role: jbsky.dashboard_graylog
      vars:
        graylog_dashboards:
          - title: "Firewall GeoIP Dashboard"
            file: "firewall-geoip.json"
          - title: "Web Traffic Dashboard"
            file: "web-traffic.json"
```

### Skip pipeline management

```yaml
- hosts: graylog.example.com
  roles:
    - role: jbsky.dashboard_graylog
      vars:
        graylog_pipelines_manage: false
      tags: [dashboards]
```

## Adding Custom Dashboards

1. Export your dashboard from Graylog UI or create the JSON manually
2. Structure: `{"dashboard": {...}, "search": {"queries": [...]}}`
3. Place the JSON in `files/dashboards/`
4. Add an entry to `graylog_dashboards` in `defaults/main.yml` or your playbook vars

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
| `/api/system/messageprocessors/config` | PUT | Processor order |

## License

MIT
