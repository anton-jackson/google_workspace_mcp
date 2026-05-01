---
name: google-workspace
description: >
  Router for Google Workspace operations across Drive, Docs, Sheets, and Calendar
  via the google-workspace MCP server. Handles authentication setup and cross-service
  workflows. Triggers on generic mentions of Google Workspace, OAuth setup, or when
  a request spans multiple services (e.g. "find a doc in Drive and edit it").
---

# Google Workspace -- Router

This skill is a thin router. For service-specific work, the dedicated skills load on their own:
- **google-drive** -- files, folders, sharing, permissions
- **google-docs** -- Google Docs content and formatting
- **google-sheets** -- spreadsheets, cells, ranges, formatting
- **google-calendar** -- events, scheduling, availability

This skill stays loaded for cross-service workflows and shared concerns (auth, common params).

## Common parameter -- always required

Every tool requires `user_google_email` (string). Ask the user once per session and reuse it. Tool calls without it fail.

## Tool name prefix

Tools are exposed by the `google-workspace` MCP server. In Claude Code the agent can call them by base name; in other harnesses prefix as needed (e.g. `google-workspace:search_drive_files`).

## First-time auth setup

If a tool returns a credential error:

1. **OAuth credentials** -- direct the user to <https://console.cloud.google.com/apis/credentials>:
   - Create OAuth 2.0 Client ID (Desktop application)
   - Enable APIs: Drive, Docs, Sheets, Calendar
   - Copy Client ID and Client Secret

2. **Store credentials** -- ask the user to add the following to their environment (do NOT ask them to paste secrets in chat):
   ```
   GOOGLE_OAUTH_CLIENT_ID=<their client id>
   GOOGLE_OAUTH_CLIENT_SECRET=<their secret>
   ```

3. **Authenticate** -- call `start_google_auth` with `user_google_email` and `service_name` (e.g. `"drive"`). Browser opens for OAuth. Credentials cache to `~/.google_workspace_mcp/credentials` for future sessions.

In most cases, just call the tool you need -- auth happens automatically on first use.

## Auth tool

### start_google_auth
Start the OAuth flow (legacy OAuth 2.0; auto-disabled when OAuth 2.1 is enabled).

| Parameter | Type | Required | Notes |
|-----------|------|----------|-------|
| user_google_email | string | no | |
| service_name | string | yes | e.g. `"drive"`, `"docs"`, `"sheets"`, `"calendar"` |

## Cross-service workflows

### Find and share a file
1. `search_drive_files` -- find by name/type
2. `manage_drive_access` with `action: "grant"` -- share it
3. `get_drive_shareable_link` -- return the link

### Open a doc found via Drive search
1. `search_drive_files` with `file_type: "document"` -- get the doc ID
2. Switch to **google-docs** skill -- use `get_doc_as_markdown` with that ID

### Open a sheet found via Drive search
1. `search_drive_files` with `file_type: "spreadsheet"` -- get the sheet ID
2. Switch to **google-sheets** skill -- use `read_sheet_values`

### Attach a Drive file to a calendar event
1. `search_drive_files` -- find the file, get the file ID or URL
2. `manage_event` with `attachments: ["<file_id_or_url>"]`

## Tips

- **`user_google_email`** stays the same across all tool calls in a session. Store it once.
- **IDs accept full URLs** -- `document_id`, `spreadsheet_id`, and `file_id` parameters all accept the full Google URL or just the ID.
- **Errors with "credentials"** -- run `start_google_auth` for the relevant service.
