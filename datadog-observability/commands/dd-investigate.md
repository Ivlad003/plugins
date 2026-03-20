---
description: Investigate a production incident using Datadog data — correlates logs, monitors, metrics, events, and infrastructure
---

# Datadog Incident Investigation

Investigate a production incident by correlating multiple Datadog signals.

## Instructions

Use the `datadog-investigator` agent to handle this investigation. Pass along any context the user provided (service name, time range, symptoms).

The agent will:
1. Check triggered monitors
2. Search error logs and aggregate by service
3. Check recent events (deployments, config changes)
4. Review infrastructure metrics
5. Correlate signals and identify probable cause
6. Present a structured incident summary
