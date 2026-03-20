---
description: Search and filter Datadog logs with any query
---

# Datadog Log Search

Search Datadog logs using the provided query.

## Instructions

1. Verify environment variables:
```bash
test -n "$DD_API_KEY" && test -n "$DD_APP_KEY" && echo "OK" || echo "MISSING: set DD_API_KEY and DD_APP_KEY"
```

2. Take the user's log query (provided as the argument to this command). If no query is provided, ask the user what they want to search for.

3. Execute the search:
```bash
curl -s -X POST "https://api.${DD_SITE:-datadoghq.com}/api/v2/logs/events/search" \
  -H 'Content-Type: application/json' \
  -H "DD-API-KEY: $DD_API_KEY" \
  -H "DD-APPLICATION-KEY: $DD_APP_KEY" \
  -d '{
    "filter": {
      "query": "USER_QUERY_HERE",
      "from": "now-1h",
      "to": "now"
    },
    "sort": "-timestamp",
    "page": { "limit": 25 }
  }' \
  | python3 -m json.tool
```

4. Present the results in a clean, readable format:
   - Show timestamp, service, status, and message for each log entry
   - Highlight error patterns if present
   - Report total count of matches
   - Don't dump raw JSON unless the user asks for it
