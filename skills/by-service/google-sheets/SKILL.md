---
name: google-sheets
description: >
  Reads, writes, and formats Google Sheets -- cell values, ranges, sheet tabs,
  conditional formatting, comments. Triggers on "update the spreadsheet", "read
  cells", "write a row", "format range", "create spreadsheet", "add a sheet",
  "conditional formatting", or any work targeting Google Sheets data or layout.
---

# Google Sheets

All tools require `user_google_email` (string). The `spreadsheet_id` parameter accepts a spreadsheet ID or a full URL.

## Tool index

| Task | Tool |
|------|------|
| List user's spreadsheets | `list_spreadsheets` |
| Get metadata (title, sheets, locale) | `get_spreadsheet_info` |
| Read cells | `read_sheet_values` |
| Write/clear cells | `modify_sheet_values` |
| Create spreadsheet | `create_spreadsheet` |
| Add a sheet (tab) | `create_sheet` |
| Move rows between sheets | `move_sheet_rows` |
| Format a range | `format_sheet_range` |
| Conditional formatting | `manage_conditional_formatting` |
| List comments | `list_spreadsheet_comments` |
| Create / reply / resolve comment | `manage_spreadsheet_comment` |

## Search & Info

### list_spreadsheets
| Parameter | Type | Required | Default |
|-----------|------|----------|---------|
| user_google_email | string | yes | |
| max_results | integer | no | 25 |

### get_spreadsheet_info
Returns title, locale, list of sheets (names + IDs).

| Parameter | Required |
|-----------|----------|
| user_google_email | yes |
| spreadsheet_id | yes |

## Read & Write

### read_sheet_values
| Parameter | Type | Required | Default | Notes |
|-----------|------|----------|---------|-------|
| user_google_email | string | yes | | |
| spreadsheet_id | string | yes | | |
| range_name | string | no | A1:Z1000 | A1 notation, e.g. `Sheet1!A1:D10` |
| include_hyperlinks | boolean | no | false | Slower |
| include_notes | boolean | no | false | Slower |

### modify_sheet_values
Write, update, or clear.

| Parameter | Type | Required | Default | Notes |
|-----------|------|----------|---------|-------|
| user_google_email | string | yes | | |
| spreadsheet_id | string | yes | | |
| range_name | string | yes | | A1 notation |
| values | array or string | conditional | | 2D array. Required unless `clear_values=true`. JSON string or list both ok |
| value_input_option | string | no | USER_ENTERED | `RAW` or `USER_ENTERED` |
| clear_values | boolean | no | false | Clear instead of write |

`USER_ENTERED` parses formulas, dates, numbers as if typed into the UI. `RAW` stores literal strings.

## Create

### create_spreadsheet
| Parameter | Type | Required | Notes |
|-----------|------|----------|-------|
| user_google_email | string | yes | |
| title | string | yes | |
| sheet_names | array of strings | no | Default: one sheet with default name |

### create_sheet
| Parameter | Required |
|-----------|----------|
| user_google_email | yes |
| spreadsheet_id | yes |
| sheet_name | yes |

### move_sheet_rows
Move rows between sheets in the same spreadsheet. Preserves formulas, types, formatting.

| Parameter | Type | Required | Notes |
|-----------|------|----------|-------|
| user_google_email | string | yes | |
| spreadsheet_id | string | yes | |
| source_sheet | string | yes | Name |
| start_row | integer | yes | 1-based, inclusive |
| end_row | integer | yes | 1-based, inclusive |
| destination_sheet | string | yes | Name |

## Formatting

### format_sheet_range
| Parameter | Type | Required | Notes |
|-----------|------|----------|-------|
| user_google_email | string | yes | |
| spreadsheet_id | string | yes | |
| range_name | string | yes | A1 notation |
| background_color | string | no | `#RRGGBB` |
| text_color | string | no | `#RRGGBB` |
| number_format_type | string | no | `NUMBER`, `CURRENCY`, `DATE`, `PERCENT`, etc. |
| number_format_pattern | string | no | Custom pattern |
| wrap_strategy | string | no | `WRAP`, `CLIP`, `OVERFLOW_CELL` |
| horizontal_alignment | string | no | `LEFT`, `CENTER`, `RIGHT` |
| vertical_alignment | string | no | `TOP`, `MIDDLE`, `BOTTOM` |
| bold / italic | boolean | no | |
| font_size | integer | no | |

### manage_conditional_formatting
| Parameter | Type | Required | Notes |
|-----------|------|----------|-------|
| user_google_email | string | yes | |
| spreadsheet_id | string | yes | |
| action | string | yes | `add`, `update`, `delete` |
| range_name | string | for add | A1 notation. Optional for update (preserves existing if omitted) |
| condition_type | string | for add | `NUMBER_GREATER`, `NUMBER_LESS`, `NUMBER_BETWEEN`, `TEXT_CONTAINS`, `TEXT_NOT_CONTAINS`, `DATE_BEFORE`, `DATE_AFTER`, `CUSTOM_FORMULA`, `BLANK`, `NOT_BLANK` |
| condition_values | array or string | conditional | Depends on `condition_type` |
| background_color | string | no | `#RRGGBB` applied when matched |
| text_color | string | no | `#RRGGBB` applied when matched |
| rule_index | integer | for update/delete | 0-based |
| gradient_points | array or string | no | Color-scale rules. Each: `{type: MIN/MAX/NUMBER/PERCENT/PERCENTILE, value, color}`. Overrides boolean styles |
| sheet_name | string | no | First sheet by default |

## Comments

### list_spreadsheet_comments
`user_google_email`, `spreadsheet_id`.

### manage_spreadsheet_comment
| Parameter | Required | Notes |
|-----------|----------|-------|
| user_google_email | yes | |
| spreadsheet_id | yes | |
| action | yes | `create`, `reply`, `resolve` |
| comment_content | for create/reply | |
| comment_id | for reply/resolve | |

## Tips

- **Range notation**: A1 throughout. Always include the sheet name for multi-sheet workbooks (`Sheet2!A1:C10`). Without a sheet name, the first sheet is used.
- **Read before write**: call `get_spreadsheet_info` first if you don't know the sheet names.
- **Verify**: after a write, `read_sheet_values` on the same range to confirm.
- **Append a row**: read to find the last used row, then `modify_sheet_values` with `range_name="Sheet1!A<n>"` and a 1-row 2D array.
- **Formulas**: write with `value_input_option="USER_ENTERED"` and prefix with `=` (e.g. `=SUM(A1:A10)`). With `RAW` the formula is stored as a literal string.
