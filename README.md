# O-Matic Server Connection — Marketplace

> **What this is:** the plugin that connects Claude Code and Codex to an O-Matic Server, and ships the Probot, Fred, and Data skills.
> **What this is NOT:** a database. It bundles no PostgreSQL and no pgvector — it's the wire and the crew, not the brain.
> The brain it connects to lives in **[o-matic-server](https://github.com/lucidIT-LLC/o-matic-server)**.

Canonical distribution for the **O-Matic Server Connection** plugin — the MCP
connection layer for an O-Matic factory. Ships the Probot (orchestrator), Fred
(storage + connection CRUD), and Data (analyst + DBA) skills.

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
