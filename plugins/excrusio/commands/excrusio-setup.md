---
description: Verify the Excrusio MCP connection and show what this token can do
allowed-tools: mcp__excrusio__whoami, mcp__excrusio__refresh_tools
---

Check the Excrusio MCP connection:

1. Call the `whoami` tool.
2. If it reports the token is invalid, tell the user to open the dashboard, go to **Settings > Integrations > API Tokens**, create a token, and re-run `claude plugin install excrusio@excrusio` (or update the `EXCRUSIO_TOKEN` plugin setting), then stop.
3. Otherwise summarise in a short table: user, role, token name and expiry, enabled modules, and the effective permissions grouped by resource.
4. List the hidden tools with their reason, so the user knows which scopes are missing.
5. End with two or three example requests the user can make with this token (e.g. "list published tours", "create a draft hotel from this brief"), based on the permissions actually present.
