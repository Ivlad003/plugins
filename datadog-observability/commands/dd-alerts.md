---
description: Check triggered Datadog monitors and alerts
---

# Datadog Alerts Check

Check for triggered monitors in Datadog.

## Instructions

1. Verify environment variables:
```bash
test -n "$DD_API_KEY" && test -n "$DD_APP_KEY" && echo "OK" || echo "MISSING: set DD_API_KEY and DD_APP_KEY"
```

2. Fetch all monitors and filter to triggered ones:
```bash
curl -s -G "https://api.${DD_SITE:-datadoghq.com}/api/v1/monitor" \
  -H "DD-API-KEY: $DD_API_KEY" \
  -H "DD-APPLICATION-KEY: $DD_APP_KEY" \
  | python3 -c "
import sys, json
monitors = json.load(sys.stdin)
triggered = [m for m in monitors if m.get('overall_state') in ('Alert', 'Warn', 'No Data')]
print(json.dumps(triggered, indent=2))
"
```

3. Present results as a summary:
   - Count of triggered monitors by state (Alert, Warn, No Data)
   - Each monitor: name, state, message, tags
   - If no triggered monitors, report "No active alerts"
