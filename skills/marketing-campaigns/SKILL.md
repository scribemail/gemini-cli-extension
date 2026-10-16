---
name: marketing-campaigns
description: Run a Scribe signature marketing campaign: build the banner content, target it, schedule/pause/resume it, and measure it. Use when the user wants to promote an event, launch, or offer through the banner under everyone's email signature.
---

# Running a signature marketing campaign

A campaign is a banner rendered beneath assigned signatures for a time window. It is created as
a **draft** and does nothing until it is scheduled.

## Build it

1. **`get_content_schema`** before authoring `content` — a campaign's content is the same
   strict block tree as a signature's `draft_content` (a `stack` root whose `children` are
   blocks). Adapt the returned `example`; do not guess the format.
2. **`create_marketing_campaign`** requires `name`, `start_time`, `end_time` (ISO-8601) and
   `time_zone` (IANA, e.g. `Europe/Paris`). Missing one → 422. Optional: `content` and the
   targeting fields `smart_fields_conditions` and `signature_template_ids`.
3. **`preview_marketing_campaign`** renders the banner to HTML with placeholder values, no
   tracking. Read it and confirm the layout before scheduling — an empty `html` means the
   campaign has no content yet.
4. **`update_marketing_campaign`** is partial (omit = unchanged, `content: null` clears).
   `status` is read-only: it only moves through the lifecycle tools below.

## Ship it

| Action                              | Tool                          | Refuses when                                                                           |
| ----------------------------------- | ----------------------------- | -------------------------------------------------------------------------------------- |
| Activate a draft or paused campaign | `schedule_marketing_campaign` | 409 if not schedulable (e.g. the end time has passed); 403 if the plan lacks campaigns |
| Pause a live campaign               | `pause_marketing_campaign`    | 409 if not active                                                                      |
| Resume a paused one                 | `resume_marketing_campaign`   | 409 if not paused; 403 on plan                                                         |
| Delete                              | `delete_marketing_campaign`   | Stops an active campaign first                                                         |

A scheduled campaign goes live immediately if `start_time` has already passed, otherwise it
waits for it. **Confirm with the user before scheduling** — this puts the banner on outgoing
company email.

## Target it

- `signature_template_ids` scopes the campaign to specific signatures.
- `smart_fields_conditions` scopes it by teammate attributes (same OR-of-ANDs shape as
  signature recipient rules).
- Leave both empty for workspace-wide.
- A signature variant can opt out of banners via `show_marketing_campaigns` on the variant.

## Measure it

Pass the campaign's id as `marketing_campaign_id` to `get_analytics_views` and
`get_analytics_clicks` for its own views, clicks, and clickthrough rate; `group_by:
"marketing_campaign"` ranks campaigns against each other. `get_analytics_events_summary`
attributes downstream conversion events. See the `signature-analytics` skill.
