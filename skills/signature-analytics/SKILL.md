---
name: signature-analytics
description: Answer questions about Scribe signature performance (views, clicks, clickthrough rate, conversion events, and identified visitors), choosing timeseries vs breakdown correctly and reporting the numbers honestly. Use for any "how is our signature performing / who clicks / what did it drive" question.
---

# Reading Scribe analytics

Every analytics tool is read-only. The whole grammar is: **timeseries by default, ranked
breakdown when you set `group_by`.**

## Views and clicks

`get_analytics_views` and `get_analytics_clicks` share one input shape:

- **Timeseries** (no `group_by`): bucketed by `granularity` — `hour` | `day` | `week` | `month`,
  UTC. Returns `{ data: [{ period, count }], totals: { count } }`. Use for trends over time.
- **Breakdown** (`group_by` = `signature_template` | `marketing_campaign` | `teammate` |
  `country`): returns ranked rows, each already carrying `view_count`, `click_count` **and**
  `clickthrough_rate`. One call answers "which signature/teammate drives the most engagement" —
  do not fetch views and clicks separately and divide them yourself.
- Filter with `start_date` / `end_date` and any of `signature_template_id`,
  `marketing_campaign_id`, `teammate_id`.

## Conversion events and visitors

- `list_analytics_events` — raw recorded events (name, value, currency, attribution, visitor
  ids), filterable by date, name, attribution, country, visitor. Paginated.
- `get_analytics_events_summary` — aggregated count and sum of `value`; timeseries by default,
  or a breakdown by `group_by` (name, signature_template, …, currency, user), rankable by
  `count` or `value`. This is where revenue attribution lives.
- `list_visitors` / `get_visitor` — people identified through the SDK's identify call, with
  traits, first/last seen, and first-touch attribution; `get_visitor` adds lifetime event stats.
- `get_visitor_breakdown` — ranks signatures or teammates by how many visitors they first
  acquired.
- `get_workspace` reports `event_tracking` — the Event Tracking ID and whether events would be
  ingested at all. Check it before concluding "we have no conversions": tracking may simply not
  be installed.

## Reporting well

- State the exact window and granularity you queried. "Last 30 days" is a choice, not a fact —
  say which dates you used.
- Zero is a real answer. If a campaign or signature has no views, report that rather than
  widening the window until a number appears, and say what you widened if you do.
- Never invent or interpolate a figure. Every number in the answer comes from a tool result.
- Prefer one breakdown call over N filtered calls; each breakdown row is already complete.
- `get_analytics_overview` and `get_analytics_breakdown` are **deprecated aliases** kept so
  stale cached tool lists keep working. Use the views/clicks tools above in new work.
