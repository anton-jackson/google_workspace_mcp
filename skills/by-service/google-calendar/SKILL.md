---
name: google-calendar
description: >
  Manages Google Calendar events and availability -- list calendars, get/search
  events, create/update/delete events, attach Drive files, add Google Meet links,
  check free/busy. Triggers on "schedule a meeting", "check my calendar", "create
  an event", "find a time", "free time", "availability", "send invite", or any
  mention of calendar events or scheduling.
---

# Google Calendar

All tools require `user_google_email` (string). Times use RFC 3339 (e.g. `2026-05-15T09:00:00-04:00` or `2026-05-15T13:00:00Z`).

## Tool index

| Task | Tool |
|------|------|
| List the user's calendars | `list_calendars` |
| Read events (list, search, single) | `get_events` |
| Create / update / delete an event | `manage_event` |
| Check free/busy | `query_freebusy` |

## list_calendars
Returns summary, ID, and primary status for each calendar.

| Parameter | Required |
|-----------|----------|
| user_google_email | yes |

## get_events
Single event by ID, list over a time range, or keyword search.

| Parameter | Type | Required | Default | Notes |
|-----------|------|----------|---------|-------|
| user_google_email | string | yes | | |
| calendar_id | string | no | primary | From `list_calendars`. Or use `"primary"` |
| event_id | string | no | | Fetch one event; ignores time filters |
| time_min | string | no | now | RFC 3339 (`2026-03-19T09:00:00Z` or date-only `2026-03-19`) |
| time_max | string | no | | RFC 3339 (exclusive) |
| max_results | integer | no | 25 | |
| query | string | no | | Keyword search across summary, description, location |
| detailed | boolean | no | false | Adds description, location, attendees with response status |
| include_attachments | boolean | no | false | Show attachment details. Only effective when `detailed=true` |

## manage_event
Create, update, or delete an event.

| Parameter | Type | Required | Notes |
|-----------|------|----------|-------|
| user_google_email | string | yes | |
| action | string | yes | `create`, `update`, `delete` |
| summary | string | for create | Event title |
| start_time | string | for create | RFC 3339 |
| end_time | string | for create | RFC 3339 |
| event_id | string | for update/delete | |
| calendar_id | string | no | Default `primary` |
| description | string | no | |
| location | string | no | |
| attendees | array | no | Email strings, or attendee objects (`{"email": "...", "optional": true}`) |
| timezone | string | no | e.g. `Australia/Melbourne` |
| attachments | array of strings | no | Drive file URLs or IDs |
| add_google_meet | boolean | no | Adds (or removes) a Meet link |
| reminders | array | no | List of dicts: `{"method": "email"|"popup", "minutes": <int>}` |
| use_default_reminders | boolean | no | |
| transparency | string | no | `opaque` (busy) or `transparent` (free) |
| visibility | string | no | `default`, `public`, `private`, `confidential` |
| color_id | string | no | "1"-"11" (update only) |
| guests_can_modify | boolean | no | |
| guests_can_invite_others | boolean | no | |
| guests_can_see_other_guests | boolean | no | |

### Reminder format
```json
[
  {"method": "email", "minutes": 30},
  {"method": "popup", "minutes": 10}
]
```

## query_freebusy
Free/busy across one or more calendars.

| Parameter | Type | Required | Notes |
|-----------|------|----------|-------|
| user_google_email | string | yes | |
| time_min | string | yes | RFC 3339 |
| time_max | string | yes | RFC 3339 |
| calendar_ids | array | no | Default: primary |
| group_expansion_max | integer | no | Max members per group (cap 100) |
| calendar_expansion_max | integer | no | Max calendars (cap 50) |

## Tips

- **Time format**: RFC 3339 throughout. Date-only strings (`2026-03-19`) parse as midnight UTC.
- **All-day events**: pass date-only strings to `start_time`/`end_time`. For a single all-day event on March 20, use `start_time="2026-03-20"` and `end_time="2026-03-21"`.
- **Calendar IDs**: discover via `list_calendars`. `primary` always works for the user's main calendar.
- **Attendees**: simple form `["alice@example.com", "bob@example.com"]`. Object form for extras: `[{"email": "alice@example.com", "optional": true}]`.
- **Attaching Drive files**: pass file IDs or full Drive URLs in `attachments`. Pair with `attachments` lookup via the Drive skill if you don't know the ID.
- **Time-zone safety**: when creating across DST or international zones, set `timezone` explicitly rather than relying on the times' offsets alone.
- **Searching**: `get_events` with `query` does substring match on summary/description/location. For finding a known event by title, this is faster than listing and filtering.
- **Move/reschedule**: use `manage_event` with `action: "update"` and set `start_time`/`end_time`. Other fields not provided are left unchanged.
