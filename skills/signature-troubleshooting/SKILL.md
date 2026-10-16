---
name: signature-troubleshooting
description: Diagnose why a Scribe signature is missing, stale, or wrong in someone's mail client by walking installation status, integration health, recipient sources, and publish state in the order that finds the cause fastest. Use for "my signature isn't showing up / is out of date / looks wrong".
---

# Why isn't this signature showing up?

Work down this list. Each step is one tool call and rules out a whole class of cause, so stop
as soon as one explains the symptom rather than gathering everything first.

1. **Is it installed for that address?**
   `list_signature_installations` with `email_address` (no need to resolve the UUID). One row
   per email × signature × integration/email client, with `status` = `installed` | `failed` |
   `uninstalled`. A `failed` row carries an `error_code` — `user_not_found` means the recipient
   has no account on that integration, which is a directory problem, not a Scribe one.
2. **Is the workspace's integration healthy?**
   `list_integrations` returns active integrations including failing ones, with `status`
   (`active` / `error`) and, when erroring, `error_message` + `error_since`. A workspace-level
   error explains failures across _everyone_ — check this before investigating one person.
3. **Are they actually a recipient?**
   `list_signature_recipients` for the template. If they are absent, nothing was ever going to
   install — this is a targeting problem; see the `signature-rollout` skill.
4. **Has the template ever been published?**
   `list_signatures` → `last_published_at`. Edits live in `draft_content` and never ship until
   `publish_signature` runs. A signature that "won't update" is usually an unpublished draft.
   Confirm by comparing `preview_signature` with `content: "draft"` vs `"published"`.
5. **What is actually installed for them?**
   `get_signature_html` with their `email_id` returns the real final HTML — live content, their
   real smart-field values, tracking on. Compare it against what they say they see. Empty means
   never published; 404/422 means that address is not a recipient.
6. **Are their details wrong rather than their signature?**
   `list_teammates` shows each smart-field value with an `integration_id`. Non-null = synced
   from a directory integration and read-only in Scribe — fix it in Google, Microsoft or Zoho
   Mail, and the value flows back. `update_teammate` silently skips integration-synced and image
   fields.

## Repair

- Correct content: fix `draft_content`, `preview_signature`, then `publish_signature`.
- Correct content but a stale or failed deployment: `install_signature` re-deploys the current
  live content without publishing anything new. 403 = plan lacks installation; 409 = no
  signature integration connected.
- Blocked outside Scribe (directory, permissions, plan): say so plainly and point at the fix.
  `search_docs` / `read_docs_page` have the setup guides — quote those rather than improvising.
- Genuinely broken or missing capability: offer `send_feedback` (`category: "bug"`) and send it
  only with the user's consent.
