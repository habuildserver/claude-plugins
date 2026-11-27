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

## Skills

| Skill | Use it for |
|---|---|
| `edit-a-screen` | Change an existing screen: text, add or remove a card or button, reorder blocks, who sees a block |
| `create-a-screen` | A new screen, or a variation of one for a group of members |
| `my-proposals` | Status, review outcome, update, roll back or withdraw your own proposals |

Claude never changes anything directly. Every change is a **proposal** a reviewer approves or rejects in the admin
panel, and an approved screen change still becomes a draft published through the normal flow. Claude acts as you, so
Habuild's own permissions decide what you can read and propose.

## Staying connected

An access token lasts 24 hours and a refresh token 30 days, the same as the admin panel. Claude refreshes on its own and
each refresh restarts the 30 days. After 30 days without use, authenticate again.

## Development environment

To try the dev gateway instead of production:

```
claude mcp add --transport http habuild-dev https://mcp-service.hb-dev.in/mcp/v1
```

Then `/mcp` → **habuild-dev** → Authenticate, and Allow on `admin-panel.hb-dev.in`.

## Requires

Gateway deployed with OAuth on (`OAUTH_ISSUER_URL`, `ADMIN_PANEL_URL`) — mcp-service PR #12 — and the
admin panel `/connect-claude` page.
