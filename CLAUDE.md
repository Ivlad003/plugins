# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A Claude Code plugin marketplace (`ivlad-plugins`) containing plugins for observability and DevOps. Currently ships one plugin: `newrelic-observability`.

## Repository Structure

```
.claude-plugin/marketplace.json   # Marketplace registry — lists all plugins with metadata
<plugin-name>/                    # Each plugin is a self-contained directory
  .claude-plugin/plugin.json      # Plugin manifest (name, version, keywords)
  commands/*.md                   # Slash commands (frontmatter + instructions)
  skills/*/SKILL.md               # Skills (auto-activated by keyword triggers)
  agents/*.md                     # Subagent definitions
  README.md                       # Plugin-specific docs
```

## Adding a New Plugin

1. Create a new directory at the repo root with the plugin name.
2. Add `.claude-plugin/plugin.json` with name, description, version, author, category, and keywords.
3. Register it in the top-level `.claude-plugin/marketplace.json` under the `plugins` array.
4. Add commands, skills, and/or agents as `.md` files following the existing patterns.
5. Update the root `README.md` with installation instructions.

## Plugin Component Conventions

- **Commands** (`commands/*.md`): Frontmatter with `description`, body contains step-by-step instructions for Claude to follow when the slash command is invoked.
- **Skills** (`skills/*/SKILL.md`): Frontmatter with `name` and `description` (including trigger keywords). Body contains full operational instructions, API patterns, and workflows.
- **Agents** (`agents/*.md`): Frontmatter with `description`. Body defines the agent's persona, protocol, and output format.

## Key Details for newrelic-observability

- All New Relic API calls use `curl` directly — no MCP server, no Python dependencies beyond `python3 -m json.tool` for formatting.
- Requires `NEW_RELIC_API_KEY` and `NEW_RELIC_ACCOUNT_ID` environment variables.
- Uses two API surfaces: NerdGraph GraphQL (`api.newrelic.com/graphql`) and REST v2 (`api.newrelic.com/v2/`).
- EU region uses `api.eu.newrelic.com/graphql`.
