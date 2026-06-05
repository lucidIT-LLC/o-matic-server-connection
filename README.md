# O-Matic Server Connection (Claude Code + Codex Plugin)

Bundled PostgreSQL + pgvector MCP server for the O-Matic factory brain, packaged for Claude Code and OpenAI Codex. Install it once per host and let each factory project route through its own `.omatic/factory.json`. Ships Probot, Fred, and Data as plugin-bundled skills.

**Version:** 1.4.1
**Author:** James Walker / O-Matic AI Research Lab

---

## What this is

The O-Matic factory brain is a Postgres database with `pgvector` on the O-Matic Server. Agents reach it through an MCP server — `omatic-server-connection` — that exposes factory tools plus raw SQL tools.

Until now that MCP server was a hand-maintained entry in `claude_desktop_config.json`. This plugin packages the same server so it installs and updates like any other plugin. No JSON surgery on the desktop config.

The server code is shared. Claude Code and Codex differ only in their plugin manifests and host bootstrap metadata.

---

## What's inside

```
omatic-server-connection/
  .claude-plugin/
    plugin.json        # Claude plugin manifest (declares skills/)
  .codex-plugin/
    plugin.json        # Codex plugin manifest (declares skills/)
  .mcp.json            # Codex MCP server config
  skills/
    omatic-server-connection/SKILL.md  # generic plugin operating guide
    orch-o-matic-probot/SKILL.md   # Probot v14.1 — orchestrator
    data-o-matic-data/SKILL.md     # Data v5.0 — analyst, architect + factory DBA
    find-o-matic-fred/SKILL.md     # Fred v9.1 — storage + connection CRUD
  server/              # bundled Node MCP server
    index.js
    connections.js
    tools.js
    package.json
    package-lock.json
    node_modules/       # runtime deps (pg, @modelcontextprotocol/sdk)
  README.md
```

The Claude manifest points the MCP server at `${CLAUDE_PLUGIN_ROOT}/server/index.js`. The Codex manifest points to `./.mcp.json`, which launches `./server/index.js` from the installed plugin root. No absolute paths are committed.

---

## How it picks a database

On Claude Code, the server receives `OMATIC_PROJECT_ROOT=${CLAUDE_PROJECT_DIR}` — the directory Claude Code was opened in. On Codex, the manifest passes `OMATIC_PROJECT_ROOT=${CODEX_WORKSPACE}` and `OMATIC_FACTORY_JSON_PATH=${CODEX_WORKSPACE}/.omatic/factory.json` when the host expands those variables. The resolver also checks common workspace env vars (`CODEX_PROJECT_ROOT`, `CODEX_WORKSPACE_ROOT`, `WORKSPACE_ROOT`, `INIT_CWD`, `PWD`) before falling back to the MCP process working directory. Cowork should use `factory_json_path` in the desktop extension config. In all cases it walks up looking for `.omatic/factory.json` and connects to the connection that file names.

Each factory project carries its own `.omatic/factory.json`:

```json
{
  "factory_id": "omatic",
  "server_name": "O-Matic",
  "platform_profile": "claude-code",
  "database_url": "postgresql://user:password@host:5432/database"
}
```

`.omatic/factory.json` is gitignored — credentials never reach the repo. A committed `.omatic/factory.json.example` template ships in each factory for fresh clones.

One plugin install serves every factory project. The project you open decides which brain you're talking to.

---

## Install In Claude Code

1. In Claude Code, add this repository as a plugin marketplace: `lucid3ye/o-matic-server`.
2. Install the **omatic-server-connection** plugin from that marketplace.
3. Restart Claude Code.
4. Approve the `omatic-server-connection` MCP server on first launch when prompted.
5. **Remove the `omatic-server` block from `claude_desktop_config.json`** — the plugin replaces it. Leaving both means two processes competing for the same server name.

## Install In OpenAI Codex

1. Add this repository as a Codex plugin marketplace.
2. Install the **omatic-server-connection** plugin.
3. Restart Codex so the plugin-managed MCP server is loaded.
4. Open Codex from a factory project containing `.omatic/factory.json`, or use `omatic_add_connection` to create/update the project connection file.
5. Verify with `omatic_resolve_factory`.

Fresh machines: run `npm install` once inside `server/` if `node_modules/` is absent. The repository currently commits `node_modules/` so the plugin can run immediately after clone/install.

---

## Tools exposed

| Tool | Purpose |
|---|---|
| `omatic_factory_startup` / `omatic_factory_health_check` | Startup surface, readiness, embedding health |
| `omatic_factory_startup_run` | Side-effecting startup runner: opens a platform session, seeds readiness, records built-in probes, warms retrieval, and returns scoped startup |
| `omatic_search_memory` | FTS-backed factory memory search |
| `omatic_embedding_status` | Reports embedding config, vector extensions, indexes, and plugin embedding certainty |
| `omatic_list_tasks` / `omatic_record_decision` / `omatic_record_session_event` / `omatic_record_probe_result` | Factory state writes |
| `omatic_resolve_factory` | Reports the active factory and resolved `factory_file` path |
| `omatic_claim_work` / `omatic_release_work` | Advisory work claims (if installed) |
| `omatic_execute_sql` | Guarded SQL — `confirm_destructive=true` required for DDL/DML |
| `o-matic-server-{factory}:execute_sql` | Raw SQL, modern name |
| `postgres-cabinet-{factory}:execute_sql` | Raw SQL, legacy alias — retained for backward compatibility |

---

## Verify after install

```
omatic_resolve_factory
```

Expect `factory_file` pointing at your project's `.omatic/factory.json` and `active_connection` matching its `factory_id`. Then run `o-matic-server-{factory}:execute_sql` with `SELECT 1` to confirm the connection is live.

---

## Changelog

- **1.4.1** — Published to `lucidIT-LLC/o-matic-server-connection` (repo renamed from `o-matic-server-plugin`).
  - **Renamed** `omatic-server` → `omatic-server-connection` across package id, MCP server registration, the generic skill, and marketplace. The plugin is the *connection*; `lucidIT-LLC/o-matic-server` is the DB image distro.
  - **Strict project-root resolver retained** (rule 259, from 1.4.0 — "no walk-up / not stuck on the first DB"). Merged with the local improvements rather than overwritten.
  - **Codex `.mcp.json` fix** — uses the spec `mcp_servers` key (was `mcpServers`; Codex silently failed to register the connector).
  - **Kernel skills regenerated** from the persona gold records: Probot 14.1, Fred 9.1, Data 5.0 (friendly-android character) — each SKILL.md stamped with its `identity_signature`.
  - Net: one plugin installs **skills + connector on both Claude Code and Codex**.
- **1.3.4** — Codex connector fix + kernel skill regeneration.
  - **`.mcp.json` now uses the Codex-spec `mcp_servers` key** (was `mcpServers`, the Claude convention). Per the OpenAI Codex plugin spec, `.mcp.json` accepts only a direct server map or a `mcp_servers` wrapper — with `mcpServers`, Codex silently failed to register the connector. Skills loaded; the connector did not.
  - **Kernel skills regenerated from the persona gold records** (factory brain): Probot 14.1.0, Fred 9.1.0, Data 5.0.0 (character replacement — friendly affable android). Each SKILL.md header now carries its `identity_signature` for drift detection.
  - Legacy `factory/closed-factory/` kernel duplicates retired; the DB gold record + this plugin are the canonical source + shipped export.
- **1.3.2** — Multi-platform startup hardening.
  - Codex manifest now passes workspace-derived `OMATIC_PROJECT_ROOT` and `OMATIC_FACTORY_JSON_PATH` when host variables are available.
  - Claude Code manifest no longer hardcodes the O-Matic project path, database URL, or Cowork platform. It uses `${CLAUDE_PROJECT_DIR}` and `OMATIC_PLATFORM=claude-code`.
  - Resolver checks multiple host workspace variables before falling back to plugin CWD and reports resolution diagnostics in `omatic_resolve_factory`.
  - Added `omatic_factory_startup_run` to create a platform-specific session, seed `session_mcp_status`, record built-in probe results, warm retrieval, and return `v_mcp_readiness_by_session` in one call.
- **1.3.1** — Hotfix on top of 1.3.0 (post-Smith audit).
  - **Defensive `${VAR}` literal detection** in `connections.js`. If a host runtime (Cowork .mcpb in some versions, certain Codex installs) fails to expand `${CLAUDE_PROJECT_DIR}` or other manifest env vars, the literal string is now treated as unset and the plugin falls back to `process.cwd()` instead of resolving to a dead path like `outputs/${CLAUDE_PROJECT_DIR}/.omatic/factory.json`.
  - **Platform precedence corrected.** `OMATIC_PLATFORM` env var now wins over `factory.json` `platform_profile`. Stale `platform_profile: "codex"` values in a shared factory.json no longer override the live surface.
  - **`ConnectionManager.reload()`.** `omatic_add_connection` and `omatic_remove_connection` now call `reload()` on the live ConnectionManager — invalidates stale pools, picks up new configs from disk, drops removed connections. The previous behavior was a no-op for the running session (false affordance Smith correctly flagged).
  - **Schema fixes:**
    - `omatic_record_session_event`: targets the actual `session_log` columns `(session_date, session_id varchar, platform, agent, event_type, detail text)`. Accepts `detail` (preferred) or `content` (legacy alias); object → JSON string. `session_id` coerced to varchar.
    - `omatic_record_probe_result`: correct arg order for `fn_record_probe_result(p_connector_id, p_session_id, p_result, p_note)` and `p_note` is text, not jsonb. Accepts `note` (preferred) or `detail` (legacy alias).
- **1.3.0** — Multi-factory rewrite.
  - **A1**: Claude Code manifest now sets `OMATIC_PROJECT_ROOT=${CLAUDE_PROJECT_DIR}` (was `${CLAUDE_PLUGIN_ROOT}`, which pointed the plugin at its own install dir and broke walk-up discovery). Codex unchanged — its CWD already resolves to the project root.
  - **A2**: Server declares `capabilities.tools.listChanged: true` and emits `notifications/tools/list_changed` after add/remove/set_active. Claude Code 2.1.0+ refreshes its tool list automatically — no restart needed.
  - **A3**: New `omatic_set_active_connection` tool switches the session's active connection without restart. Between-task only.
  - **A4**: Per-connection variants of base tools (`omatic_factory_startup:selife`, `omatic_execute_sql:thenest`, etc.) pin calls to a specific configured connection regardless of active default. Unsuffixed names still hit the default.
  - **A5**: Cowork `.mcpb` extension and Claude Code / Codex plugin now share one source. `omatic-server-connection/{connections,tools,index}.js` are copies of `plugins/omatic-server/server/*`. No more drift.
  - **A6**: Cowork extension gains a `factory_json_path` user_config field. Set it to an absolute path to an existing `.omatic/factory.json` and the extension reads connections from that file, bypassing the Desktop UI fields. Bridge between Cowork and Claude Code / Codex project configs.
  - **A8**: `writeFactoryConfig` now writes atomically (temp file + rename). Prevents lost updates from concurrent worktrees or surfaces.
  - **A9**: Upgrade migration — on first boot after upgrade, if no `.omatic/factory.json` is found AND `OMATIC_DATABASE_URL` env is set (legacy hardcoded fallback), the plugin writes one from the env DSN. Refuses to write into a plugin install dir.
  - **Skills bundled**: Probot v14 (orchestrator), Data v4.0 (analyst + factory DBA), Fred v9.0 (storage + connection CRUD) ship in `skills/`. Cowork `.mcpb` does NOT bundle skills (MCPB spec has no `skills` field) — Cowork operators continue installing the anthropic-skills suite for Probot/Data/Fred.
  - **Governance**: Rule 237 (skills not agents — authored skill prose), rule 238 (plugin distribution boundary — skills bundle with tool surface), rule 239 (plugin bootstrap pointer — never `${CLAUDE_PLUGIN_ROOT}`) persisted to the factory DB.
- **1.2.0** — Added OpenAI Codex plugin manifest, Codex marketplace metadata, Codex skill instructions, and `omatic_embedding_status`.
- **1.1.0** — Packaged as a Claude Code plugin. Server reads `database_url` from per-project `.omatic/factory.json`. Modern `o-matic-server-{factory}` tool name added alongside the legacy `postgres-cabinet-*` alias. Replaces the `claude_desktop_config.json` entry.
