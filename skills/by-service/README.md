# Google Workspace MCP -- Per-Service Skills

Five separate Claude Code skills for the `google-workspace` MCP server, split by service so each loads only when its triggers fire.

## Bundle contents

```
google-workspace/    -- thin router; auth setup; cross-service workflows
google-drive/        -- files, folders, sharing, permissions
google-docs/         -- Docs content + layout workflow guidance
google-sheets/       -- spreadsheets, ranges, formatting
google-calendar/     -- events, scheduling, free/busy
```

Each `SKILL.md` is self-contained with parameter tables inlined -- no separate `references/` folder. That makes the bundle work in any harness that consumes a single SKILL.md per skill.

## Install (Claude Code)

User-level (all projects):
```bash
cp -r google-workspace google-drive google-docs google-sheets google-calendar ~/.claude/skills/
```

Or project-level:
```bash
mkdir -p .claude/skills
cp -r google-workspace google-drive google-docs google-sheets google-calendar .claude/skills/
```

You can install only the services you need -- the router (`google-workspace`) is helpful but optional.

## Install (claude.ai consumer apps)

If your Claude.ai plan supports custom skill upload, zip each directory individually and upload them as separate skills.

## Prerequisites

The `google-workspace` MCP server must be configured and reachable. See the project README at the repo root for setup. These skills only document how to use the tools -- they don't provide them.

## Notes on the split

- **Triggers are scoped by keyword**: Drive owns "file/folder/share/permission", Docs owns "document/paragraph/heading", Sheets owns "spreadsheet/cell/range", Calendar owns "event/meeting/schedule".
- **Cross-service workflows** (e.g. find a doc in Drive, then edit it) are documented in the `google-workspace` router skill.
- **`user_google_email`** is required by every tool. The router covers auth setup; service skills assume auth is already done.
- **No Gmail, Slides, Forms, Tasks, Contacts, Chat, Apps Script, or Search** -- add them later from the upstream `references/` if needed.
