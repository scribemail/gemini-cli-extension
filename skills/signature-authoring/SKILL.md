---
name: signature-authoring
description: Create or edit a Scribe email-signature template: the draft → preview → publish → install loop, the block-tree content format, smart fields, and reply/internal variants. Use whenever the user wants to build, restyle, fix, or publish a signature in Scribe.
---

# Authoring a Scribe signature template

A signature template in Scribe has **two copies of its content**: `draft_content` (what you
edit) and `content` (what is live in people's mail clients). Writing never touches the live
copy — only `publish_signature` promotes draft → live. Preview before you publish.

## The loop

1. **`get_content_schema` first.** Always, before writing any `draft_content`. One call returns
   the strict JSON Schema for the block tree, a flat `enums` lookup of every allowed value
   (including the 100+ `icon` names), a `social_icons` map from platform name to its
   non-obvious enum (`linkedin` → `linkedin-in`, `facebook` → `facebook-f`), a complete worked
   `example` to adapt, and short authoring `guidance`. Never guess enum spellings or
   brute-force the format — the server strict-validates and rejects unknown keys.
2. **`list_smart_fields`** to see the merge fields available (job title, phone, photo, …). Put
   smart fields in the tree rather than hardcoding one person's details — that is what makes
   one template work for the whole company. `kind` and `integration_id` matter: a field with a
   non-null `integration_id` is synced from Google, Microsoft or Zoho Mail and is read-only per
   teammate.
3. **`list_assets`** for logos and banners already in the workspace before uploading a new one
   with `create_asset` (`file_base64` + `filename`, or `source_url`; images only, ≤15 MB).
4. **`create_signature`** (`name` required, `draft_content` the block tree, optional
   `folder_id`) or **`update_signature`** for an existing one. Updates are partial: an omitted
   field is unchanged, `null` clears it. The live signature does not move.
5. **`preview_signature`** — renders the draft to HTML with smart fields shown as their labels.
   **Read the returned HTML.** Check for overlapping lines, broken spacing, a missing logo, a
   social row that collapsed. Fix the draft and preview again. This is the only cheap feedback
   loop; publishing is what people actually see.
6. **`publish_signature`** — copies draft → live, sets `last_published_at`, and triggers
   installation where the plan and integration allow it. Confirm with the user first: this
   changes the signature on real outgoing mail.
7. **`get_signature_html`** with a specific `email_id` for the real, final installed HTML for
   one recipient (live content, that person's real values, tracking on). Empty until the
   template has been published, and the email must actually be a recipient.

## Variants (reply / internal)

`list_signature_variants` shows the **reply** and **internal** (same-domain) alternates, each
with its own state and content, plus a `supported` flag. Variants require an active Microsoft
365 signatures integration — without one, writes fail with 409 `variants_unavailable`.

- `update_signature_variant` upserts a variant's `draft_content` and its
  `activated` / `hidden` / `show_marketing_campaigns` flags.
- `publish_signature_variant` promotes that variant's draft and redeploys via the primary.
- `delete_signature_variant` is a **reset to primary**, not a hard delete: it deactivates the
  variant and restores the primary's content.

## Rules of thumb

- Reads run freely; **confirm before every write** — create, update, publish, install, delete.
- Publishing is the moment of truth. Preview first, every time, even for a one-word change.
- `install_signature` re-deploys the current live `content` without publishing a new draft —
  use it when a rollout failed, not as a substitute for publish. 403 = the plan lacks
  installation; 409 = no signature integration connected.
- Organize with folders (`list_folders` / `create_folder`); a folder with signatures in it
  cannot be deleted (409).
- For product questions ("how do smart fields work?", "how do I set up DNS?"), call
  `search_docs` / `read_docs_page` instead of answering from memory.
