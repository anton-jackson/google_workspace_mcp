---
name: google-docs
description: >
  Reads, creates, and edits Google Docs -- text content, paragraph styles, headings,
  lists, tables, images, comments, headers/footers, tabs, and PDF export. Triggers on
  "edit a doc", "create a Google Doc", "add a heading", "find and replace in doc",
  "insert a table", "format a paragraph", "export doc to PDF", or any work targeting
  Google Docs content (not Drive file management).
---

# Google Docs

All tools require `user_google_email` (string). The `document_id` parameter accepts a doc ID or a full URL.

## Tool index

| Task | Tool |
|------|------|
| Read as Markdown (preferred) | `get_doc_as_markdown` |
| Read as plain text | `get_doc_content` |
| Search docs by name | `search_docs` |
| List docs in folder | `list_docs_in_folder` |
| Create new doc | `create_doc` |
| Insert/replace text + character formatting | `modify_doc_text` |
| Find and replace | `find_and_replace_doc` |
| Paragraph style / heading / list / alignment | `update_paragraph_style` |
| Insert table / list / page break | `insert_doc_elements` |
| Insert table pre-filled with data | `create_table_with_data` |
| Insert image | `insert_doc_image` |
| Headers and footers | `update_doc_headers_footers` |
| Manage tabs (create/rename/delete/populate) | `manage_doc_tab` |
| Export to PDF (saved to Drive) | `export_doc_to_pdf` |
| Inspect structure (indices, tabs, tables) | `inspect_doc_structure` |
| Inspect a single table | `debug_table_structure` |
| Multiple ops atomically | `batch_update_doc` |
| List comments | `list_document_comments` |
| Create / reply / resolve comment | `manage_document_comment` |

## Reading

### get_doc_as_markdown -- preferred for reading
Preserves headings, bold/italic/strikethrough, links, code spans, nested lists, tables.

| Parameter | Type | Required | Default | Notes |
|-----------|------|----------|---------|-------|
| user_google_email | string | yes | | |
| document_id | string | yes | | |
| include_comments | boolean | no | true | |
| comment_mode | string | no | inline | `inline`, `appendix`, `none` |
| include_resolved | boolean | no | false | |

### get_doc_content
Plain text only. Use this for non-Google-Docs files (.docx etc.) or when you don't need formatting.

| Parameter | Required |
|-----------|----------|
| user_google_email | yes |
| document_id | yes (doc ID or file ID) |

### search_docs / list_docs_in_folder

| Tool | Params |
|------|--------|
| `search_docs` | `user_google_email` (yes), `query` (yes), `page_size` (no, default 10) |
| `list_docs_in_folder` | `user_google_email` (yes), `folder_id` (no, default root), `page_size` (no, default 100) |

## Creating

### create_doc
| Parameter | Required | Notes |
|-----------|----------|-------|
| user_google_email | yes | |
| title | yes | |
| content | no | Initial body text |

## Text editing

### modify_doc_text
Insert/replace text and/or apply character formatting in one call.

| Parameter | Type | Required | Notes |
|-----------|------|----------|-------|
| user_google_email | string | yes | |
| document_id | string | yes | |
| start_index | integer | yes | From `inspect_doc_structure`. `0` is also accepted as "first writable body position" |
| end_index | integer | no | If omitted, inserts at `start_index`. Provide both to replace a range |
| text | string | no | Insert/replace content. Omit to format existing text |
| bold / italic / underline / strikethrough | boolean | no | |
| font_size | integer | no | Points |
| font_family | string | no | e.g. "Arial" |
| text_color | string | no | `#RRGGBB` |
| background_color | string | no | `#RRGGBB` (highlight) |
| link_url | string | no | Hyperlink |

### find_and_replace_doc
| Parameter | Required | Notes |
|-----------|----------|-------|
| user_google_email | yes | |
| document_id | yes | |
| find_text | yes | |
| replace_text | yes | |
| match_case | no | default false |
| tab_id | no | Target one tab |

## Paragraph styling

### update_paragraph_style
| Parameter | Type | Required | Notes |
|-----------|------|----------|-------|
| user_google_email | string | yes | |
| document_id | string | yes | |
| start_index | integer | yes | First char of the paragraph |
| end_index | integer | yes | Should cover the whole paragraph (exclusive) |
| heading_level | integer | no | 0=NORMAL, 1-6=H1-H6 |
| named_style_type | string | no | `NORMAL_TEXT`, `TITLE`, `SUBTITLE`, `HEADING_1`..`HEADING_6`. Mutually exclusive with `heading_level` |
| alignment | string | no | `START`, `CENTER`, `END`, `JUSTIFIED` |
| line_spacing | number | no | 1.0 single, 2.0 double |
| indent_first_line / indent_start / indent_end | number | no | Points (36 = 0.5") |
| space_above / space_below | number | no | Points |
| list_type | string | no | `UNORDERED` (bullets), `ORDERED` (numbers), `NONE` (remove list) |
| list_nesting_level | integer | no | 0-8, default 0 |

## Structural elements

### insert_doc_elements
Insert a table, list, or page break.

| Parameter | Type | Required | Notes |
|-----------|------|----------|-------|
| user_google_email | string | yes | |
| document_id | string | yes | |
| element_type | string | yes | `table`, `list`, `page_break` |
| index | integer | yes | 0-based |
| rows | integer | for table | |
| columns | integer | for table | |
| list_type | string | for list | `UNORDERED` or `ORDERED` |
| text | string | no | Initial list item text |

### create_table_with_data
Pre-populated table in one call. Always run `inspect_doc_structure` first; use `total_length` as the index.

| Parameter | Type | Required | Notes |
|-----------|------|----------|-------|
| user_google_email | string | yes | |
| document_id | string | yes | |
| table_data | array | yes | 2D list of strings. All rows same column count. Use `""` not null for empty cells |
| index | integer | yes | Use `total_length` from inspect |
| bold_headers | boolean | no | Default true |
| tab_id | string | no | |

### insert_doc_image
| Parameter | Type | Required | Notes |
|-----------|------|----------|-------|
| user_google_email | string | yes | |
| document_id | string | yes | |
| image_source | string | yes | Drive file ID OR public URL |
| index | integer | yes | |
| width / height | integer | no | Points; 0 = auto |

## Headers / footers / export

### update_doc_headers_footers
| Parameter | Required | Notes |
|-----------|----------|-------|
| user_google_email | yes | |
| document_id | yes | |
| section_type | yes | `header` or `footer` |
| content | yes | Text |
| header_footer_type | no | `DEFAULT`, `FIRST_PAGE_ONLY`, `EVEN_PAGE` |

### export_doc_to_pdf
| Parameter | Required | Default |
|-----------|----------|---------|
| user_google_email | yes | |
| document_id | yes | |
| pdf_filename | no | original + "_PDF" |
| folder_id | no | root |

## Tabs

### manage_doc_tab
| Parameter | Required | Notes |
|-----------|----------|-------|
| user_google_email | yes | |
| document_id | yes | |
| action | yes | `create`, `rename`, `delete`, `populate_from_markdown` |
| tab_id | rename/delete/populate | From `inspect_doc_structure` |
| title | create/rename | |
| index | create | 0-based among siblings |
| parent_tab_id | no | Nest under a parent (create only) |
| markdown_text | populate | Markdown source |
| replace_existing | no | Default true |

`populate_from_markdown` supports headings, bold/italic/code, links, lists, code blocks, blockquotes, horizontal rules. Tables -> plain text fallback. Images -> linked alt text.

## Comments

### list_document_comments
`user_google_email`, `document_id`.

### manage_document_comment
| Parameter | Required | Notes |
|-----------|----------|-------|
| user_google_email | yes | |
| document_id | yes | |
| action | yes | `create`, `reply`, `resolve` |
| comment_content | for create/reply | |
| comment_id | for reply/resolve | |

## Inspection

### inspect_doc_structure -- call before any index-based op
Returns `total_elements`, `total_length` (max safe insertion index), `tables`, `table_details`, `tabs`.

| Parameter | Required | Default |
|-----------|----------|---------|
| user_google_email | yes | |
| document_id | yes | |
| detailed | no | false |
| tab_id | no | main doc |

### debug_table_structure
| Parameter | Required | Default |
|-----------|----------|---------|
| user_google_email | yes | |
| document_id | yes | |
| table_index | no | 0 |

## Batch

### batch_update_doc
Atomic multi-op call. `operations` is a list of dicts each with a `type`. All ops accept optional `tab_id`.

| Type | Required | Optional |
|------|----------|----------|
| `insert_text` | `index`, `text` | |
| `delete_text` | `start_index`, `end_index` | |
| `replace_text` | `start_index`, `end_index`, `text` | |
| `format_text` | `start_index`, `end_index` | bold, italic, underline, strikethrough, font_size, font_family, text_color, background_color, link_url |
| `update_paragraph_style` | `start_index`, `end_index` | heading_level, alignment, line_spacing, indents, spacing |
| `insert_table` | `index`, `rows`, `columns` | |
| `insert_page_break` | `index` | |
| `find_replace` | `find_text`, `replace_text` | match_case |
| `create_bullet_list` | `start_index`, `end_index` | list_type (`UNORDERED`/`ORDERED`/`NONE`), nesting_level (0-8), paragraph_start_indices |
| `insert_doc_tab` | `title`, `index` | parent_tab_id |
| `delete_doc_tab` | `tab_id` | |
| `update_doc_tab` | `tab_id`, `title` | |

When mixing inserts/deletes, **process from highest index to lowest** so earlier ops don't invalidate later indices.

---

# Layout Workflow -- avoid Google Docs API formatting pitfalls

The Docs API has well-known issues that bite agents. Follow this pattern when building a formatted doc.

## Why this matters

- **Style cascade**: a heading and body inserted in the same block both inherit the heading style.
- **Index shifting**: every insert moves character positions; subsequent ops target the wrong text.
- **List merging**: list items inserted one at a time become separate lists.
- **Table cell indexing**: filling cells top-to-bottom corrupts positions; go bottom-right -> top-left.

## Core principle: build incrementally

Insert small batches. Apply styles immediately. Re-read between batches.

## Step 1 -- Start point
- Branded template? `copy_drive_file` then `find_and_replace_doc` for placeholders.
- Otherwise `create_doc`.
- Use single-tab docs unless you pass `tab_id` to every `update_paragraph_style` call.

## Step 2 -- The batch pattern (per content group)
1. Insert heading text alone (one paragraph, ending `\n`)
2. Apply heading style with `update_paragraph_style` (`heading_level=1/2/3`)
3. Insert body text (one or more paragraphs, each ending `\n`)
4. Apply `named_style_type="NORMAL_TEXT"` to body
5. Re-read with `get_doc_content` for fresh positions

### Critical rules

| Rule | Why |
|------|-----|
| Never insert heading + body in the same block | Both inherit heading style |
| Always reset to `NORMAL_TEXT` after a heading's body | Next insert otherwise inherits the heading style |
| Re-read after every insertion | Positions shift; stale indices = wrong styling |
| Lists: insert ALL items in ONE block | Separate inserts produce separate lists |

## Step 3 -- Lists
1. Insert all items as one block: `"Item 1\nItem 2\nItem 3\n"`
2. Apply `list_type="UNORDERED"` (or `ORDERED`) to the entire range in ONE call
3. Insert the next paragraph and reset to `NORMAL_TEXT` to break the list
4. For ORDERED lists: don't include "1." prefixes in the text -- the API renumbers automatically

## Step 4 -- Tables
1. Read doc to find current end position
2. `insert_doc_elements(element_type="table", index=<position>, rows=N, columns=M)`
3. Fill cells **bottom-right to top-left** (prevents index drift)
4. Re-read before continuing -- tables insert invisible structural characters
5. To find position after a table, try inserting at index 99999; the error reveals the actual end

## Step 5 -- Verify
**Always export to PDF and read it before claiming the doc is done.**
```
export_doc_to_pdf(document_id="<id>")
```
Check: no body styled as heading, lists render correctly, all cells filled, heading hierarchy right, no stray empties.

## Common pitfalls

| Symptom | Cause | Fix |
|---------|-------|-----|
| Body text in heading font | Heading + body in same block, or no NORMAL_TEXT reset | Insert separately, apply NORMAL_TEXT to body |
| All paragraphs become bullets | `list_type` applied too widely | Limit range to actual list items |
| "1. 1. MAP" | Text has "1." AND list_type=ORDERED | Remove number prefixes when using ORDERED |
| `€` shows literally | Unicode escape not resolved | Use the actual `€` character |
| Empty table cells | Index shift while filling | Fill bottom-right -> top-left, re-read between |
| Wrong tab styled | `tab_id` missing on `update_paragraph_style` | Pass `tab_id` or use single-tab doc |

---

## Tips

- **Indices**: Index 0 is the leading section break. `inspect_doc_structure` reports body content starting at 1. `modify_doc_text` and `update_paragraph_style` accept `start_index=0` as "first writable position." After any edit that adds/removes text, indices shift -- re-inspect or work end-to-start.
- **Format without changing text**: call `modify_doc_text` with `start_index` + `end_index` and omit `text`.
- **Reading**: prefer `get_doc_as_markdown`. Use `get_doc_content` only for plain text or non-Docs files.
- **Tabs**: discover tabs via `inspect_doc_structure` (no `tab_id`). Then pass `tab_id` to scope edits.
