# Codex Adapter

Codex uses:

- Plugin manifest: `.codex-plugin/plugin.json`
- MCP manifest: `.mcp.json`
- Server entrypoint: `server/index.js`
- Bundled skills: `skills/*/SKILL.md`

Install selector:

```text
omatic-server-connection@lucidIT-LLC
```

Full support requires a project `.omatic/factory.json` or a connection created with the plugin connection tools.

After updating the source plugin, reinstall from `@lucidIT-LLC` and start a fresh Codex thread so the MCP server and skill list reload.
