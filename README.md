# Habuild plugin for coding agents

**habuild-mcp** connects your coding agent to every onboarded Habuild service through one MCP endpoint
(`prefix__tool`, e.g. `ux__list_surfaces`), and adds three skills:

| Skill | Use it for |
|---|---|
| `edit-a-screen` | Change an existing screen: text, add or remove a card or button, reorder blocks, who sees a block |
| `create-a-screen` | A new screen, or a variation of one for a group of members |
| `my-proposals` | Status, review outcome, update, roll back or withdraw your own proposals |

Endpoint: `https://mcp-service.habuild.in/mcp/v1` (streamable HTTP, OAuth 2.1 with PKCE and dynamic client
registration).

## How access works

Installing the plugin gives you nothing on its own. The first time your agent uses the server it opens your
browser on the Habuild admin panel's **Connect** page. Sign in with your admin-panel account and click **Allow**.
The agent then acts as you: an `INTERNAL` role or higher is required, and every Habuild service still applies its
own permissions. Nothing in this repository is a secret; there is no token to paste anywhere.

Claude never changes anything directly. Every change is a **proposal** a reviewer approves or rejects in the admin
panel, and an approved screen change still becomes a draft published through the normal flow.

Access lasts while you keep using it: an access token for 24 hours and a refresh token for 30 days, renewed on
each refresh. After 30 days without use, sign in again.

## Installation

Install separately for each agent you use. After installing, sign in once (the "Sign in" line under each agent).

### Claude Code

```
/plugin marketplace add habuildserver/claude-plugins
/plugin install habuild-mcp@habuild
```

Sign in: `/mcp` → **habuild** → Authenticate.

### Claude Desktop and claude.ai

An organization Owner adds a custom connector (Organization settings → Connectors) with the endpoint URL above.
Each member then clicks **Connect** and signs in.

### Codex (CLI and app)

```
codex plugin marketplace add habuildserver/claude-plugins
codex plugin add habuild-mcp@habuild
```

Or in the TUI, `/plugins`. Sign in: the app prompts on install; in the CLI run `codex mcp list` to see the server
name, then `codex mcp login <name>`.

### GitHub Copilot CLI

```
copilot plugin install habuildserver/claude-plugins
```

Sign in: `/mcp auth habuild`.

### Gemini CLI

```
gemini extensions install https://github.com/habuildserver/claude-plugins
```

Sign in: automatic on first use, or `/mcp auth habuild`. Update later with `gemini extensions update habuild-mcp`.

### Qwen Code

```
qwen extensions install habuildserver/claude-plugins
```

Pick `habuild-mcp` if prompted. Sign in: `/mcp` → **habuild** → Auth.

### Antigravity

```
agy plugin install https://github.com/habuildserver/claude-plugins
```

Sign in: Antigravity opens the browser when the server first connects. Reinstall with the same command to update.

### Cursor

Teams and Enterprise: Dashboard → Plugins → **Import from Repo** with this repository. Otherwise copy the repository
to `~/.cursor/plugins/local/habuild-mcp` and reload the window, or add the server by hand in `~/.cursor/mcp.json`:

```json
{ "mcpServers": { "habuild": { "url": "https://mcp-service.habuild.in/mcp/v1" } } }
```

Sign in: in Cursor's MCP settings, use the sign-in button on **habuild**.

### Devin CLI

```
devin plugins install habuildserver/claude-plugins
```

Sign in: prompted on first use, or `devin mcp login <name>` (`devin mcp list` shows the name).

### Factory Droid

```
droid plugin marketplace add https://github.com/habuildserver/claude-plugins
droid plugin install habuild-mcp@claude-plugins
```

`droid plugin marketplace list` shows the marketplace name if it differs. Sign in: `/mcp` → **habuild** → Authenticate.

### Grok Build CLI

```
grok plugin install habuildserver/claude-plugins --trust
```

Sign in: the browser opens on first use, or `/mcps` → **habuild** → `i`.

### Kimi Code

```
/plugins install https://github.com/habuildserver/claude-plugins
```

Sign in: `/mcp-config login <name>` (`/mcp-config` lists the name).

### Muse

```
muse plugins marketplace add habuild https://github.com/habuildserver/claude-plugins
muse plugins install habuild-mcp@habuild
muse plugins approve habuild-mcp
```

Sign in: `muse mcp login habuild`.

### OpenCode

Add to `opencode.json`:

```json
{ "mcp": { "habuild": { "type": "remote", "url": "https://mcp-service.habuild.in/mcp/v1" } } }
```

(OpenCode V2: `{ "mcp": { "servers": { "habuild": { "type": "remote", "url": "…" } } } }`.) Sign in:
`opencode mcp auth habuild`.

### Pi

Add to `~/.pi/agent/mcp.json`:

```json
{ "mcpServers": { "habuild": { "url": "https://mcp-service.habuild.in/mcp/v1" } } }
```

Sign in: `/mcp login habuild` inside a session.

### Hermes Agent

Hermes plugins cannot carry an OAuth MCP server, so add it to `~/.hermes/config.yaml`:

```yaml
mcp_servers:
  habuild:
    url: "https://mcp-service.habuild.in/mcp/v1"
    auth: oauth
```

Sign in: `hermes mcp login habuild`.

### Any other MCP client

Point it at the endpoint URL as a streamable-HTTP server with OAuth. A client that cannot do remote servers or OAuth
can use the stdio bridge, which opens the same browser sign-in:

```json
{ "mcpServers": { "habuild": { "command": "npx", "args": ["-y", "mcp-remote", "https://mcp-service.habuild.in/mcp/v1"] } } }
```

## Development environment

To try the dev gateway instead of production, add the server by hand with the dev URL, e.g. in Claude Code:

```
claude mcp add --transport http habuild-dev https://mcp-service.hb-dev.in/mcp/v1
```

Then sign in as above; the Connect page is on `admin-panel.hb-dev.in`.

## Which file each agent reads

| File | Agents |
|---|---|
| `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json` (`"source": "./"`), `.mcp.json` | Claude Code, Codex, Copilot, Devin, Droid, Grok, Muse (marketplace) |
| `.codex-plugin/plugin.json` | Codex |
| `.cursor-plugin/plugin.json` | Cursor |
| `gemini-extension.json` (`httpUrl`) | Gemini CLI, Qwen Code |
| `.antigravity-plugin/plugin.json`, `mcp_config.json` (`serverUrl`) | Antigravity |
| `.devin-plugin/plugin.json` | Devin |
| `.kimi-plugin/plugin.json` | Kimi Code |
| `.muse-plugin/plugin.json` | Muse |
| `skills/` | every agent with plugin skills |

The shapes differ on purpose: `.mcp.json` needs `"type": "http"` (Claude Code reads an entry without it as stdio),
`gemini-extension.json` needs `httpUrl` (Qwen reads a plain `url` as SSE), and Antigravity reads only
`mcp_config.json`. Keep the URL and `version` the same in every file.

## Gateway requirements

Each agent signs in with its own redirect address. Local CLIs return to `http://localhost` or `http://127.0.0.1`,
which the gateway always accepts. Hosted callbacks must be in the gateway's `OAUTH_REDIRECT_URI_ALLOWLIST`:

| Agent | Callback to allowlist |
|---|---|
| claude.ai, Claude Desktop | `https://claude.ai/api/mcp/auth_callback`, `claude://claude.ai/mcp-auth-callback/sdk` |
| Cursor (desktop too: it registers both its callbacks at once) | `https://www.cursor.com/agents/mcp/oauth/callback` |
| Antigravity | to confirm against dev: `https://antigravity.google/oauth-callback` or `https://antigravity.google.com/oauth/callback` |
| Devin Cloud, Factory web sessions (optional) | `https://api.devin.ai/mcp/oauth/callback`, `https://app.factory.ai/mcp/oauth/callback` |

Requires the gateway with OAuth on (mcp-service PR #12) and the admin panel `/connect-claude` page
(admin-panel-fe-remix PR #1057).
