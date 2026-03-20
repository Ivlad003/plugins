# ivlad-plugins

Claude Code plugin marketplace by ivlad003 — observability, DevOps, and developer productivity tools.

## Installation

```bash
/plugin marketplace add Ivlad003/plugins
```

## Available Plugins

### newrelic-observability

Query and analyze New Relic data directly from Claude Code via REST API calls. No MCP server or Python needed.

```bash
/plugin install newrelic-observability@ivlad-plugins
```

**Prerequisites:** Set `NEW_RELIC_API_KEY` and `NEW_RELIC_ACCOUNT_ID` environment variables.

**Includes:**

| Component | Description |
|-----------|-------------|
| `/nr-query <NRQL>` | Run any NRQL query |
| `/nr-alerts` | Check active alert violations |
| `/nr-status` | Health overview of all monitored apps |
| `/nr-investigate` | Full incident investigation with root cause analysis |
| `newrelic-ops` skill | 12 API operations + 3 investigation workflows |
| `newrelic-investigator` agent | Correlates signals for incident root cause analysis |

**Usage:**
```
"What's the error rate for my-app?"
"Are there any open alerts?"
/nr-status
/nr-query SELECT average(duration) FROM Transaction FACET name SINCE 1 hour ago
/nr-investigate checkout is returning 500s
```

## License

MIT
