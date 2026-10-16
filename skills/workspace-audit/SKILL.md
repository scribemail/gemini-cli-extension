---
name: workspace-audit
description: Sweep a Scribe workspace for problems in one pass (teammates with no signature, failed installations, erroring integrations, unpublished drafts, expired campaigns) and report what to fix first. Use for "audit our signatures", "who's missing a signature", or a health check before a rollout.
---

# Auditing a Scribe workspace

A read-only sweep. Run the checks, then report **findings ranked by how many people they
affect**, each with the one action that fixes it. Do not make changes as part of an audit —
propose them.

## The sweep

1. **Workspace and plan** — `get_workspace`: `plan` (tier, status, billing interval),
   `seat_count` (what is invoiced), and `event_tracking`. A past-due plan or a plan without
   installation explains failures that look technical.
2. **Integration health** — `list_integrations`: any `status: "error"` row, with
   `error_message` and `error_since`. This is the highest-leverage finding — a broken
   integration silently blocks every install behind it.
3. **Coverage** — `list_emails` (with `auto_paginate`) against `list_signature_recipients` for
   each signature from `list_signatures`. Addresses in the first set and none of the second have
   **no signature at all**. Report them grouped by teammate, not as a flat list of aliases.
4. **Installation failures** — `list_signature_installations` with `status: "failed"`. Group by
   `error_code`: a wall of `user_not_found` is one directory problem, not fifty tickets.
5. **Unpublished work** — signatures whose `last_published_at` is null or older than their last
   edit have drafts nobody can see. Flag them; publishing is the user's call.
6. **Campaigns** — `list_marketing_campaigns`: drafts that were never scheduled, and active
   campaigns whose `end_time` has passed.
7. **Field hygiene** _(optional, when the ask is broader)_ — `list_smart_fields` plus
   `list_teammates` to spot fields that are empty for most of the team, which render as blank
   lines in every signature.

## Reporting

Lead with the count and the fix, not the method:

> **3 integrations erroring** — Microsoft 365 has been failing since 12 Aug
> (`invalid_grant`); 41 of 48 failed installs trace to it. Reconnect it first.
>
> **7 teammates with no signature** — Ana, Ben, … Assign them the "Company Default"
> template, or set `assign_to_all_future_emails` on it so new joiners are covered.
>
> **2 unpublished drafts** — "Sales EMEA" was edited 3 days ago and never published.

Keep it short enough to act on. Offer to fix the top item, and get explicit confirmation before
any write. For product questions raised by the findings, use `search_docs` / `read_docs_page`
rather than answering from memory.
