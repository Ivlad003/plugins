# Datadog Plugin for Claude Code

Query and analyze Datadog observability data directly from Claude Code using direct REST API calls. No MCP server or extra dependencies needed.

## Prerequisites

Set these environment variables before using:

```bash
export DD_API_KEY="your-api-key"
export DD_APP_KEY="your-application-key"
export DD_SITE="datadoghq.com"  # optional, defaults to datadoghq.com
```

## Installation

### From GitHub

```bash
/plugin marketplace add ivlad003/plugins
/plugin install datadog-observability@ivlad-plugins
```

### From local directory

```bash
claude plugin install ./plugin/datadog-observability
```

## What's Included

### Slash Commands

| Command | Description |
|---------|-------------|
| `/dd-logs <query>` | Search and filter Datadog logs |
| `/dd-query <metric>` | Run a metrics query |
| `/dd-alerts` | Check triggered monitors |
| `/dd-status` | Quick health overview of infrastructure |
| `/dd-investigate` | Full incident investigation with root cause analysis |

### Skill: `datadog-ops`

Automatically activated when you ask about Datadog data. Gives Claude knowledge of 12 API operations:

1. Search logs
2. Aggregate logs by facets
3. Query metrics
4. List monitors (alerts)
5. Get monitor details
6. List events
7. List hosts
8. Host totals
9. Search APM traces
10. Get dashboard details
11. List SLOs
12. Get service summary (APM)

Plus 3 investigation workflows: Incident, Log Investigation, and Performance Analysis.

### Agent: `datadog-investigator`

A specialized agent for production incident investigation. Correlates logs, monitors, metrics, events, and infrastructure to find root causes. Produces structured incident summaries.

## Usage Examples

```
# Direct questions (skill activates automatically)
"Search Datadog logs for errors in the payment service"
"What monitors are currently triggered?"
"Show me CPU usage across all hosts"
"Any deployment events in the last 24 hours?"

# Slash commands
/dd-status
/dd-alerts
/dd-logs service:checkout status:error
/dd-query avg:system.cpu.user{env:production} by {host}
/dd-investigate the checkout service is returning 500s
```

## How It Works

This plugin instructs Claude to call Datadog APIs directly using `curl`:
- **REST v1**: `https://api.datadoghq.com/api/v1/` for metrics, monitors, events, hosts
- **REST v2**: `https://api.datadoghq.com/api/v2/` for log search and aggregation

No Python, no MCP server, no extra processes — just `curl` and your API keys.

## License

MIT License
