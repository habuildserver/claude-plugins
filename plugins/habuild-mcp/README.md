# habuild-mcp

One connector for every onboarded Habuild MCP service (`prefix__tool`, e.g. `ux__list_surfaces`).
No token to paste: Claude signs you in through the admin panel.

## Install (Claude Code)

```
/plugin marketplace add habuildserver/claude-plugins
/plugin install habuild-mcp@habuild
```

Then `/mcp` → **habuild** → Authenticate. A browser opens the admin panel; log in if needed and click
**Allow**. Stays connected as long as you use it once every 30 days.

## Staging

```
claude mcp add --transport http habuild-staging https://<staging-mcp-host>/mcp/v1
```

## Requires

Gateway deployed with OAuth on (`OAUTH_ISSUER_URL`, `ADMIN_PANEL_URL`) — mcp-service PR #12 — and the
admin panel `/connect-claude` page.
