# O-Matic Server Connection — Marketplace

Canonical distribution for the **O-Matic Server Connection** plugin — a bundled
PostgreSQL + pgvector MCP server for the O-Matic factory brain. Ships the Probot
(orchestrator), Fred (storage + connection CRUD), and Data (analyst + DBA) skills.

This repo is a **plugin marketplace** that works on both surfaces.

## Install

**Claude Code**
```
/plugin marketplace add lucidIT-LLC/o-matic-server-connection
/plugin install omatic-server-connection@omatic
```

**OpenAI Codex**
```
/plugin marketplace add lucidIT-LLC/o-matic-server-connection
```
Or via the **Add marketplace** dialog: set Source to the GitHub URL, Git ref to
`main`, and leave Sparse paths empty.

## Layout

```
.claude-plugin/marketplace.json     Claude Code marketplace manifest
.agents/plugins/marketplace.json    Codex marketplace manifest
plugins/
  omatic-server-connection/         the plugin itself
    .claude-plugin/plugin.json
    .codex-plugin/plugin.json
    .mcp.json  server/  skills/  commands/
```

— [o-matic.io](https://o-matic.io)
