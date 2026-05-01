---
name: google-drive
description: >
  Manages Google Drive files and folders -- search, list, read content, create, copy,
  move, and control sharing/permissions. Triggers on "find a file", "share a doc",
  "upload to Drive", "create folder", "set permissions", "shareable link", or any
  mention of Google Drive files, folders, or sharing.
---

# Google Drive

All tools require `user_google_email` (string). The `file_id` parameter accepts a Drive file ID or a full URL.

## Tool index

| Task | Tool |
|------|------|
| Search files/folders | `search_drive_files` |
| List items in folder | `list_drive_items` |
| Read file content as text | `get_drive_file_content` |
| Download file / get URL | `get_drive_file_download_url` |
| Create a file | `create_drive_file` |
| Create a folder | `create_drive_folder` |
| Copy a file | `copy_drive_file` |
| Update file metadata / move | `update_drive_file` |
| Set link sharing | `set_drive_file_permissions` |
| Grant / revoke / batch share | `manage_drive_access` |
| Inspect permissions | `get_drive_file_permissions` |
| Get shareable link | `get_drive_shareable_link` |
| Check public access | `check_drive_file_public_access` |
| Import to Google Doc | `import_to_google_doc` |

## Search & Browse

### search_drive_files
Search across My Drive and shared drives.

| Parameter | Type | Required | Default | Notes |
|-----------|------|----------|---------|-------|
| user_google_email | string | yes | | |
| query | string | yes | | Drive query syntax: `name contains 'foo'`, `mimeType = '...'`, `'<id>' in parents`, `modifiedTime > '2026-01-01T00:00:00'`, `trashed = false`, `sharedWithMe`. Combine with `and`/`or`/`not`. |
| page_size | integer | no | 10 | |
| page_token | string | no | | Pagination |
| drive_id | string | no | | Shared drive ID |
| include_items_from_all_drives | boolean | no | true | |
| corpora | string | no | | `user`, `domain`, `drive`, `allDrives`. Defaults to `drive` when `drive_id` set |
| file_type | string | no | | Friendly: `folder`, `document`, `spreadsheet`, `presentation`, `form`, `pdf`, `csv`, `script`, `shortcut` -- or raw MIME |
| detailed | boolean | no | true | Size, mtime, link |
| order_by | string | no | | See sort keys below |

### list_drive_items
List children of a folder.

| Parameter | Type | Required | Default | Notes |
|-----------|------|----------|---------|-------|
| user_google_email | string | yes | | |
| folder_id | string | no | root | Folder ID. Use shared drive ID for its root |
| page_size | integer | no | 100 | |
| page_token | string | no | | |
| drive_id | string | no | | |
| include_items_from_all_drives | boolean | no | true | |
| corpora | string | no | | `user`, `drive`, `allDrives` |
| file_type | string | no | | Same friendly names as search |
| detailed | boolean | no | true | |
| order_by | string | no | | |

**Sort keys for `order_by`**: `createdTime`, `folder`, `modifiedByMeTime`, `modifiedTime`, `name`, `name_natural`, `quotaBytesUsed`, `recency`, `sharedWithMeTime`, `starred`, `viewedByMeTime`. Append ` desc` for descending. Combine with commas: `folder,modifiedTime desc`.

## Content & Download

### get_drive_file_content
Get file content as text. Google Docs/Sheets/Slides export to text/CSV; Office files (.docx/.xlsx/.pptx) parse for text.

| Parameter | Type | Required | Notes |
|-----------|------|----------|-------|
| user_google_email | string | yes | |
| file_id | string | yes | |

### get_drive_file_download_url
Download to disk (stdio mode) or get a 1-hour URL (HTTP mode).

| Parameter | Type | Required | Notes |
|-----------|------|----------|-------|
| user_google_email | string | yes | |
| file_id | string | yes | |
| export_format | string | no | `pdf`, `docx`, `xlsx`, `csv`, `pptx`. Defaults: Docs->PDF, Sheets->XLSX, Slides->PDF |

## Create & Modify

### create_drive_file
| Parameter | Type | Required | Default | Notes |
|-----------|------|----------|---------|-------|
| user_google_email | string | yes | | |
| file_name | string | yes | | |
| content | string | no | | Inline content |
| folder_id | string | no | root | Parent |
| mime_type | string | no | text/plain | |
| fileUrl | string | no | | Fetch from URL (file://, http://, https://) |

### create_drive_folder
| Parameter | Type | Required | Default |
|-----------|------|----------|---------|
| user_google_email | string | yes | |
| folder_name | string | yes | |
| parent_folder_id | string | no | root |

### copy_drive_file
| Parameter | Type | Required | Default |
|-----------|------|----------|---------|
| user_google_email | string | yes | |
| file_id | string | yes | |
| new_name | string | no | "Copy of [original]" |
| parent_folder_id | string | no | root |

### update_drive_file
Update metadata, move between folders, star, trash.

| Parameter | Type | Required | Notes |
|-----------|------|----------|-------|
| user_google_email | string | yes | |
| file_id | string | yes | |
| name | string | no | |
| description | string | no | |
| mime_type | string | no | |
| add_parents | string | no | Comma-separated folder IDs |
| remove_parents | string | no | Comma-separated folder IDs |
| starred | boolean | no | |
| trashed | boolean | no | |
| writers_can_share | boolean | no | |
| copy_requires_writer_permission | boolean | no | |
| properties | object | no | Custom key-value |

To **move a file**: set both `add_parents` (destination) and `remove_parents` (source).

### import_to_google_doc
Convert MD/DOCX/TXT/HTML/RTF/ODT into a new Google Doc.

| Parameter | Type | Required | Default | Notes |
|-----------|------|----------|---------|-------|
| user_google_email | string | yes | | |
| file_name | string | yes | | |
| content | string | no | | For MD/TXT/HTML |
| file_path | string | no | | Local path; `file://` URLs ok |
| file_url | string | no | | http/https |
| source_format | string | no | auto | `md`, `markdown`, `docx`, `txt`, `html`, `rtf`, `odt` |
| folder_id | string | no | root | |

## Permissions & Sharing

### set_drive_file_permissions
Quick link-sharing toggle.

| Parameter | Type | Required | Notes |
|-----------|------|----------|-------|
| user_google_email | string | yes | |
| file_id | string | yes | |
| link_sharing | string | no | `off`, `reader`, `commenter`, `writer` |
| writers_can_share | boolean | no | |
| copy_requires_writer_permission | boolean | no | |

### manage_drive_access
All permission ops via `action`.

| Parameter | Type | Required | Notes |
|-----------|------|----------|-------|
| user_google_email | string | yes | |
| file_id | string | yes | |
| action | string | yes | `grant`, `grant_batch`, `update`, `revoke`, `transfer_owner` |
| share_with | string | no | Email/domain (for `grant`); omit for "anyone" |
| role | string | no | `reader`, `commenter`, `writer`. Default `reader` for grant |
| share_type | string | no | `user` (default), `group`, `domain`, `anyone` |
| permission_id | string | for update/revoke | From `get_drive_file_permissions` |
| recipients | array | for grant_batch | `[{email, role?, share_type?, expiration_time?}]`. Use `domain` instead of `email` for domain shares |
| send_notification | boolean | no | Default true |
| email_message | string | no | Custom note |
| expiration_time | string | no | RFC 3339, e.g. `2026-12-31T00:00:00Z` |
| allow_file_discovery | boolean | no | For domain/anyone shares |
| new_owner_email | string | for transfer_owner | |
| move_to_new_owners_root | boolean | no | Default false |

### get_drive_file_permissions
Returns metadata + all permissions (with IDs needed for update/revoke).

| Parameter | Required |
|-----------|----------|
| user_google_email | yes |
| file_id | yes |

### get_drive_shareable_link
Returns the shareable link and current sharing status.

| Parameter | Required |
|-----------|----------|
| user_google_email | yes |
| file_id | yes |

### check_drive_file_public_access
Search by name, return whether public link sharing is on.

| Parameter | Required |
|-----------|----------|
| user_google_email | yes |
| file_name | yes |

## Shared Drives -- important limitations

Files in Shared Drives are owned by the drive itself, not users. **Owner-based queries do NOT work**:
- `'user@example.com' in owners` returns nothing
- `ownedByMe = true` skips Shared Drive items
- `owners` / `ownerNames` fields are not populated; `ownedByMe` is always false

Use time-based queries instead: `modifiedTime > '2026-01-01T00:00:00'` with `order_by='modifiedTime desc'`.

Other field gaps in Shared Drives: `permissions` not in list output (use `get_drive_file_permissions`), `shared` always true, `folderColorRgb` not supported, `writersCanShare` ignored.

## Tips

- **Pagination**: search/list responses include `next_page_token`. Pass it as `page_token` for the next page. Search results are incomplete without it.
- **Permission workflow**: call `get_drive_file_permissions` to get IDs, then `manage_drive_access` with `action: "update"` or `"revoke"`.
- **Batch sharing**: `manage_drive_access` with `action: "grant_batch"` and a `recipients` list shares with many people in one call.
- **Link sharing shortcut**: `set_drive_file_permissions` with `link_sharing` flips "anyone with the link" without dealing with permission IDs.
- **Shared drives**: set `drive_id` to scope. When set, `corpora` defaults to `drive`. For folder ops in a shared drive, use a folder ID inside it (or the drive ID itself for root).
