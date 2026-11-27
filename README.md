# claude-plugins

Habuild's agent plugins. Today one: **habuild-mcp** — every onboarded Habuild MCP service
(`prefix__tool`, e.g. `ux__list_surfaces`) through a single endpoint, with sign-in through the admin
panel. No token to paste anywhere.

Endpoint: `https://mcp-service.habuild.in/mcp/v1` (streamable HTTP, OAuth 2.1 + PKCE + dynamic client
registration).

## Claude Code

```
/plugin marketplace add habuildserver/claude-plugins
/plugin install habuild-mcp@habuild
```

Then `/mcp` → **habuild** → Authenticate.

## Claude Desktop / claude.ai

Settings → Connectors → **Add custom connector** → the endpoint URL above. (In an org, an Owner adds it
under Organization settings → Connectors.)

## Cursor

Add to `~/.cursor/mcp.json` (or `.cursor/mcp.json` for one project) — same content as
[`mcp.json`](mcp.json) in this repo:

```json
{
  "mcpServers": {
    "habuild": { "url": "https://mcp-service.habuild.in/mcp/v1" }
  }
}
```

Settings → MCP → **Needs Login** → sign in. Cursor desktop's callback is loopback and works as-is;
**Cursor web and Cursor Agents** need `https://www.cursor.com/agents/mcp/oauth/callback` added to the
gateway's `OAUTH_REDIRECT_URI_ALLOWLIST` first.

## Codex CLI

Append [`codex-config.toml`](codex-config.toml) to `~/.codex/config.toml`:

```toml
experimental_use_rmcp_client = true

[mcp_servers.habuild]
url = "https://mcp-service.habuild.in/mcp/v1"
```

Then `codex mcp login habuild`, and `/mcp` in the TUI to check.

## Any other MCP client

Clients that read a `mcpServers` map take [`mcp.json`](mcp.json) as-is. One that cannot do remote
servers or OAuth gets there through the stdio bridge:

```json
{
  "mcpServers": {
    "habuild": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp-service.habuild.in/mcp/v1"]
    }
  }
}
```

The bridge opens the same browser sign-in and caches the tokens under `~/.mcp-auth`.

## Requires

Gateway deployed with OAuth on (`OAUTH_ISSUER_URL`, `ADMIN_PANEL_URL`) — mcp-service PR #12 — and the
admin panel `/connect-claude` page.
