---
description: Run a Datadog metrics query and display results
---

# Datadog Metrics Query

Run a Datadog metrics query.

## Instructions

1. Verify environment variables:
```bash
test -n "$DD_API_KEY" && test -n "$DD_APP_KEY" && echo "OK" || echo "MISSING: set DD_API_KEY and DD_APP_KEY"
```

2. Take the user's metrics query (provided as the argument to this command). If no query is provided, ask the user what metric they want to query.

3. Execute the query:
```bash
curl -s -G "https://api.${DD_SITE:-datadoghq.com}/api/v1/query" \
  -H "DD-API-KEY: $DD_API_KEY" \
  -H "DD-APPLICATION-KEY: $DD_APP_KEY" \
  --data-urlencode "query=USER_METRICS_QUERY_HERE" \
  --data-urlencode "from=$(date -v-1H +%s)" \
  --data-urlencode "to=$(date +%s)" \
  | python3 -m json.tool
```

4. Present the results in a clean, readable format (table or summary). Don't dump raw JSON unless the user asks for it.
