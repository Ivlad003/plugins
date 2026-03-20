---
description: Get a quick health overview of your Datadog-monitored infrastructure
---

# Datadog Status Overview

Get a quick health check across your Datadog-monitored infrastructure.

## Instructions

1. Verify environment variables:
```bash
test -n "$DD_API_KEY" && test -n "$DD_APP_KEY" && echo "OK" || echo "MISSING: set DD_API_KEY and DD_APP_KEY"
```

2. Run these queries to build the overview:

**Host totals:**
```bash
curl -s -G "https://api.${DD_SITE:-datadoghq.com}/api/v1/hosts/totals" \
  -H "DD-API-KEY: $DD_API_KEY" \
  -H "DD-APPLICATION-KEY: $DD_APP_KEY" \
  | python3 -m json.tool
```

**Triggered monitors:**
```bash
curl -s -G "https://api.${DD_SITE:-datadoghq.com}/api/v1/monitor" \
  -H "DD-API-KEY: $DD_API_KEY" \
  -H "DD-APPLICATION-KEY: $DD_APP_KEY" \
  | python3 -c "
import sys, json
monitors = json.load(sys.stdin)
triggered = [m for m in monitors if m.get('overall_state') in ('Alert', 'Warn', 'No Data')]
print(json.dumps({'total_monitors': len(monitors), 'triggered': len(triggered), 'details': triggered}, indent=2))
"
```

**Error log volume (last 30 min):**
```bash
curl -s -X POST "https://api.${DD_SITE:-datadoghq.com}/api/v2/logs/analytics/aggregate" \
  -H 'Content-Type: application/json' \
  -H "DD-API-KEY: $DD_API_KEY" \
  -H "DD-APPLICATION-KEY: $DD_APP_KEY" \
  -d '{
    "filter": {
      "query": "status:error",
      "from": "now-30m",
      "to": "now"
    },
    "compute": [{ "aggregation": "count" }],
    "group_by": [{ "facet": "service", "limit": 10, "sort": { "aggregation": "count", "order": "desc" } }]
  }' \
  | python3 -m json.tool
```

**CPU across hosts (last 30 min):**
```bash
curl -s -G "https://api.${DD_SITE:-datadoghq.com}/api/v1/query" \
  -H "DD-API-KEY: $DD_API_KEY" \
  -H "DD-APPLICATION-KEY: $DD_APP_KEY" \
  --data-urlencode "query=avg:system.cpu.user{*} by {host}" \
  --data-urlencode "from=$(date -v-30M +%s)" \
  --data-urlencode "to=$(date +%s)" \
  | python3 -m json.tool
```

3. Present a concise status dashboard:

```
## System Status

### Infrastructure
- Total hosts: [count] active, [count] up

### Monitors
- [count] triggered out of [total] (or "All OK")
- List triggered monitors if any

### Error Logs (last 30 min)
| Service | Error Count |
|---------|-------------|

### Host CPU
| Host | Avg CPU % |
|------|-----------|
```
