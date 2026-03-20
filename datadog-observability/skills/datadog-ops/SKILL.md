---
name: datadog-ops
description: Use when the user asks about Datadog data — logs, metrics, monitors, alerts, events, hosts, APM traces, dashboards, or production incidents. Calls Datadog APIs directly via curl. Triggers on keywords like "datadog", "dd", "monitors", "log search", "metrics query", "apm traces", "host map", "events stream".
---

# Datadog Operations

Call Datadog REST APIs directly via `curl`. No MCP server or extra dependencies needed.

## Prerequisites

Three environment variables must be set:
- `DD_API_KEY` — Datadog API key
- `DD_APP_KEY` — Datadog Application key (required for reading data)
- `DD_SITE` — (optional) Datadog site, defaults to `datadoghq.com`. Common values: `datadoghq.com`, `datadoghq.eu`, `us3.datadoghq.com`, `us5.datadoghq.com`, `ap1.datadoghq.com`

Before the first API call in a session, verify they exist:
```bash
test -n "$DD_API_KEY" && test -n "$DD_APP_KEY" && echo "OK" || echo "MISSING: set DD_API_KEY and DD_APP_KEY"
```

If missing, ask the user to set them. Do NOT proceed without them.

## Base URL

All endpoints use: `https://api.${DD_SITE:-datadoghq.com}`

## Auth Headers

All requests must include:
```
-H "DD-API-KEY: $DD_API_KEY" \
-H "DD-APPLICATION-KEY: $DD_APP_KEY"
```

## Core Operations

### 1. Search Logs (most common)

```bash
curl -s -X POST "https://api.${DD_SITE:-datadoghq.com}/api/v2/logs/events/search" \
  -H 'Content-Type: application/json' \
  -H "DD-API-KEY: $DD_API_KEY" \
  -H "DD-APPLICATION-KEY: $DD_APP_KEY" \
  -d '{
    "filter": {
      "query": "service:my-service status:error",
      "from": "now-1h",
      "to": "now"
    },
    "sort": "-timestamp",
    "page": {
      "limit": 25
    }
  }' \
  | python3 -m json.tool
```

**Common log query syntax:**
| Filter | Example |
|--------|---------|
| By service | `service:my-service` |
| By status | `status:error`, `status:warn`, `status:info` |
| By host | `host:web-01` |
| By tag | `env:production`, `version:2.1` |
| Free text | `"connection timeout"` |
| Exclude | `-status:debug` |
| Combine | `service:api status:error "timeout"` |
| Wildcard | `service:payment*` |
| Facet | `@http.status_code:500` |

**Time range formats:**
- Relative: `now-15m`, `now-1h`, `now-1d`, `now-1w`
- Absolute ISO: `2024-01-15T10:00:00Z`

### 2. Search Logs with Aggregation

```bash
curl -s -X POST "https://api.${DD_SITE:-datadoghq.com}/api/v2/logs/analytics/aggregate" \
  -H 'Content-Type: application/json' \
  -H "DD-API-KEY: $DD_API_KEY" \
  -H "DD-APPLICATION-KEY: $DD_APP_KEY" \
  -d '{
    "filter": {
      "query": "status:error",
      "from": "now-1h",
      "to": "now"
    },
    "compute": [
      { "aggregation": "count" }
    ],
    "group_by": [
      { "facet": "service", "limit": 10, "sort": { "aggregation": "count", "order": "desc" } }
    ]
  }' \
  | python3 -m json.tool
```

**Aggregation types:** `count`, `avg`, `sum`, `min`, `max`, `pc75`, `pc90`, `pc95`, `pc99`

**Common group_by facets:** `service`, `host`, `status`, `@http.status_code`, `@http.url_details.path`

### 3. Query Metrics

```bash
curl -s -G "https://api.${DD_SITE:-datadoghq.com}/api/v1/query" \
  -H "DD-API-KEY: $DD_API_KEY" \
  -H "DD-APPLICATION-KEY: $DD_APP_KEY" \
  --data-urlencode "query=avg:system.cpu.user{*} by {host}" \
  --data-urlencode "from=$(date -v-1H +%s)" \
  --data-urlencode "to=$(date +%s)" \
  | python3 -m json.tool
```

**Common metrics:**
| Metric | What |
|--------|------|
| `system.cpu.user` | Host CPU usage |
| `system.mem.used` | Memory used |
| `system.disk.used` | Disk used |
| `system.load.1` | Load average (1 min) |
| `trace.servlet.request.hits` | APM request count |
| `trace.servlet.request.duration` | APM request duration |
| `trace.servlet.request.errors` | APM error count |

**Metric query syntax:**
- `avg:metric{tag:value}` — average
- `sum:metric{tag:value}` — sum
- `max:metric{tag:value} by {host}` — max grouped by host
- `metric{service:web,env:prod}` — multiple tag filters

**Time range on macOS:**
- Last hour: `from=$(date -v-1H +%s)`, `to=$(date +%s)`
- Last day: `from=$(date -v-1d +%s)`
- Last 30 min: `from=$(date -v-30M +%s)`

**Time range on Linux:**
- Last hour: `from=$(date -d '1 hour ago' +%s)`, `to=$(date +%s)`

### 4. List Monitors (Alerts)

```bash
curl -s -G "https://api.${DD_SITE:-datadoghq.com}/api/v1/monitor" \
  -H "DD-API-KEY: $DD_API_KEY" \
  -H "DD-APPLICATION-KEY: $DD_APP_KEY" \
  | python3 -m json.tool
```

**Filter by status (triggered only):**
```bash
curl -s -G "https://api.${DD_SITE:-datadoghq.com}/api/v1/monitor" \
  -H "DD-API-KEY: $DD_API_KEY" \
  -H "DD-APPLICATION-KEY: $DD_APP_KEY" \
  --data-urlencode "monitor_tags=env:production" \
  | python3 -c "
import sys, json
monitors = json.load(sys.stdin)
triggered = [m for m in monitors if m.get('overall_state') in ('Alert', 'Warn', 'No Data')]
print(json.dumps(triggered, indent=2))
"
```

**Monitor states:** `OK`, `Alert`, `Warn`, `No Data`, `Skipped`

### 5. Get Monitor Details

```bash
curl -s -G "https://api.${DD_SITE:-datadoghq.com}/api/v1/monitor/MONITOR_ID" \
  -H "DD-API-KEY: $DD_API_KEY" \
  -H "DD-APPLICATION-KEY: $DD_APP_KEY" \
  --data-urlencode "group_states=all" \
  | python3 -m json.tool
```

### 6. List Events

```bash
curl -s -G "https://api.${DD_SITE:-datadoghq.com}/api/v1/events" \
  -H "DD-API-KEY: $DD_API_KEY" \
  -H "DD-APPLICATION-KEY: $DD_APP_KEY" \
  --data-urlencode "start=$(date -v-1d +%s)" \
  --data-urlencode "end=$(date +%s)" \
  | python3 -m json.tool
```

**Filter events by source or tag:**
```bash
  --data-urlencode "sources=deploy,chef" \
  --data-urlencode "tags=env:production"
```

### 7. List Hosts

```bash
curl -s -G "https://api.${DD_SITE:-datadoghq.com}/api/v1/hosts" \
  -H "DD-API-KEY: $DD_API_KEY" \
  -H "DD-APPLICATION-KEY: $DD_APP_KEY" \
  --data-urlencode "count=100" \
  | python3 -m json.tool
```

### 8. Host Totals

```bash
curl -s -G "https://api.${DD_SITE:-datadoghq.com}/api/v1/hosts/totals" \
  -H "DD-API-KEY: $DD_API_KEY" \
  -H "DD-APPLICATION-KEY: $DD_APP_KEY" \
  | python3 -m json.tool
```

### 9. Search APM Traces (via logs)

APM trace data can be queried through log search using trace facets:

```bash
curl -s -X POST "https://api.${DD_SITE:-datadoghq.com}/api/v2/logs/events/search" \
  -H 'Content-Type: application/json' \
  -H "DD-API-KEY: $DD_API_KEY" \
  -H "DD-APPLICATION-KEY: $DD_APP_KEY" \
  -d '{
    "filter": {
      "query": "source:trace service:my-service @http.status_code:500",
      "from": "now-1h",
      "to": "now"
    },
    "page": { "limit": 10 }
  }' \
  | python3 -m json.tool
```

### 10. Get Dashboard Details

```bash
curl -s -G "https://api.${DD_SITE:-datadoghq.com}/api/v1/dashboard/DASHBOARD_ID" \
  -H "DD-API-KEY: $DD_API_KEY" \
  -H "DD-APPLICATION-KEY: $DD_APP_KEY" \
  | python3 -m json.tool
```

**List all dashboards:**
```bash
curl -s -G "https://api.${DD_SITE:-datadoghq.com}/api/v1/dashboard" \
  -H "DD-API-KEY: $DD_API_KEY" \
  -H "DD-APPLICATION-KEY: $DD_APP_KEY" \
  | python3 -m json.tool
```

### 11. List SLOs

```bash
curl -s -G "https://api.${DD_SITE:-datadoghq.com}/api/v1/slo" \
  -H "DD-API-KEY: $DD_API_KEY" \
  -H "DD-APPLICATION-KEY: $DD_APP_KEY" \
  | python3 -m json.tool
```

### 12. Get Service Summary (APM)

```bash
curl -s -G "https://api.${DD_SITE:-datadoghq.com}/api/v1/query" \
  -H "DD-API-KEY: $DD_API_KEY" \
  -H "DD-APPLICATION-KEY: $DD_APP_KEY" \
  --data-urlencode "query=avg:trace.servlet.request.duration{service:SERVICE_NAME}" \
  --data-urlencode "from=$(date -v-1H +%s)" \
  --data-urlencode "to=$(date +%s)" \
  | python3 -m json.tool
```

## Investigation Workflows

### Incident Investigation
1. Check triggered monitors (operation 4)
2. Search error logs around incident time (operation 1)
3. Aggregate errors by service (operation 2)
4. Check recent deployment events (operation 6)
5. Review host infrastructure metrics (operation 3)
6. Correlate signals and identify root cause

### Log Investigation
1. Search for error logs: `service:TARGET status:error`
2. Aggregate by error pattern: group_by `@error.kind` or `@error.message`
3. Find affected endpoints: group_by `@http.url_details.path`
4. Check timeline: use `TIMESERIES` compute with interval
5. Correlate with deploy events (operation 6)

### Performance Analysis
1. Request duration: `avg:trace.servlet.request.duration{service:NAME}`
2. Error rate: `sum:trace.servlet.request.errors{service:NAME}.as_rate()`
3. Throughput: `sum:trace.servlet.request.hits{service:NAME}.as_rate()`
4. Host resources: `avg:system.cpu.user{host:NAME}`, `avg:system.mem.used{host:NAME}`
5. Compare with baseline using different time ranges

## Output Handling

- Always pipe curl output through `| python3 -m json.tool` for readable formatting
- For extracting just log messages:
  ```bash
  | python3 -c "import sys,json; d=json.load(sys.stdin); [print(e.get('attributes',{}).get('message','')) for e in d.get('data',[])]"
  ```
- Present results to the user in a clean table or summary — don't dump raw JSON unless asked

## Error Handling

| Error | Cause | Fix |
|-------|-------|-----|
| 403 Forbidden | Invalid API key or App key | Check `DD_API_KEY` and `DD_APP_KEY` |
| 400 Bad Request | Malformed query or invalid parameters | Check query syntax |
| 429 Too Many Requests | Rate limited | Wait and retry, reduce query scope |
| Empty results | No data matching filter in time range | Widen time range or loosen filter |
| Connection refused | Wrong site or network issue | Check `DD_SITE` value and connectivity |
