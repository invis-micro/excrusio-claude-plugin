# Excrusio plugin for Claude Code

Manage the Excrusio travel CMS from Claude Code. The plugin bundles an MCP server that talks to the Excrusio API with **your own personal access token**, so Claude can only do what your dashboard role (and the token's scopes) allow.

## 1. Create a token

1. Sign in to your Excrusio dashboard and open **Settings > Integrations > API Tokens**.
2. Click **New token**, give it a name, pick an expiry and either *All my permissions* or a custom scope set (presets: Read-only, Content editing).
3. Copy the token once. It starts with `excr_pat_` and is never shown again.

Tokens are hashed at rest, expire automatically, are revoked when your password or role changes, and can be revoked any time from the same page. Admins see and revoke tokens per user on the user edit page and in **Audit Log**.

## 2. Install the plugin

```bash
claude plugin marketplace add invis-micro/excrusio-claude-plugin
claude plugin install excrusio@excrusio
```

Claude Code asks for:

| setting | value |
|---|---|
| `EXCRUSIO_API_URL` | your dashboard origin, e.g. `https://dashboard.example.com` |
| `EXCRUSIO_TOKEN` | the token from step 1 (stored in the secure credential store) |
| `EXCRUSIO_UPLOAD_ROOT` | optional folder that `media_upload` may read from |

Then run `/excrusio-setup` inside Claude Code to verify the connection and see what the token can do.

To get a newer version later, run `claude plugin marketplace update excrusio` and reinstall.

### Without the plugin (any MCP client)

The server is the single file `server/index.js` inside the plugin folder; it needs Node.js 20 or newer and no `npm install`:

```bash
node <plugin folder>/server/index.js
```

with the same environment variables. For Claude Code manually:

```bash
claude mcp add --transport stdio excrusio \
  -e EXCRUSIO_API_URL=https://... -e EXCRUSIO_TOKEN=excr_pat_... \
  -- node <plugin folder>/server/index.js
```

## 3. What you get

| group | tools |
|---|---|
| system | `whoami`, `refresh_tools`, `dashboard_stats`, `dashboard_activity` |
| content (tour, hotel, transfer, flight, blog) | `content_list/get/create/update/trash/restore/delete_permanent/bulk_status/bulk_trash/bulk_restore/translate_preview/translations` |
| taxonomy | `categories_*`, `activities_*`, `amenities_*` |
| locations | `locations_*`, `countries_*`, `cities_*` |
| media | `media_list/get/upload/update/trash/restore/delete_permanent`, `media_duplicates`, `media_reprocess_status/start`, `media_folders_list`, `media_folder_create` |
| bookings | `bookings_list/get/update_status` (tour, hotel, transfer) |
| reviews | `reviews_list/get/update_status/delete` |
| admin (read-only) | `users_list`, `users_get`, `roles_list`, `permissions_list` |
| settings & languages | `settings_get` (secrets masked), `settings_update` (non-secret keys), `languages_*` |
| importer & analytics | `importer_jobs_list/job_get/job_preview/job_run/job_cancel`, `analytics_summary`, `analytics_top_pages` (need the Google services plugin) |

Tools you lack permission for are hidden; `whoami` lists them with the missing scope. Resources: `excrusio://me`, `excrusio://permissions`, `excrusio://settings/public`, `excrusio://schema/{type}`. Prompts: `create-content-from-brief`, `review-pending-bookings`, `publish-checklist`.

## Security model

- The token can never exceed the live permissions of your role; demoting a user instantly narrows every token they own.
- Tokens cannot create users, change roles, mint other tokens, change passwords, or edit secret/module settings. Those stay dashboard-only.
- Destructive tools (permanent delete, bulk operations, settings changes, uploads, booking status) require a confirmation in Claude Code even in auto mode.
- Every mutation is written to the server audit log with the token prefix and IP.
- The server only talks `https` (except localhost), never follows redirects, and redacts tokens from its logs.
- `media_upload` only reads allow-listed image/video files inside the upload root and rejects symlinks. A file identical or similar to existing media is not stored unless `allow_duplicate: true` is passed; the result names the existing items instead.

## License

The plugin is proprietary software of Excrusio, provided for use with an Excrusio installation you are authorised to access. See [LICENSE](LICENSE) for the terms. The bundled server contains open-source components listed, with their licenses, in [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).

Issues and questions: <https://github.com/invis-micro/excrusio-claude-plugin/issues>.
