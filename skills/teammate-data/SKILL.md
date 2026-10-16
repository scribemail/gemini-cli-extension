---
name: teammate-data
description: Bulk-manage the people behind the signatures in Scribe (smart-field values, adding or removing teammates and alias emails, and the field definitions themselves), including which writes silently do nothing because an integration owns the field. Use for "update everyone's job title / phone / office address", onboarding or offboarding, or adding a new merge field.
---

# Managing teammate data

Smart fields are the **definitions** (the merge fields a signature renders); each teammate holds
a **value** per field. Bulk edits go wrong in two ways that look nothing alike, and one of them
returns success — read the write-safety section before running a batch.

## Who owns the data

Two layers of ownership, both of which can block a write:

- **The team list.** If a Google, Microsoft or Zoho Mail directory integration drives teammates,
  `create_teammate`, `delete_teammate`, and every email write return **409**. That is not an error
  to route around — the directory is the source of truth. Change it there and it syncs back.
- **The individual field.** A smart field with a non-null `integration_id` syncs its values from an
  integration and is read-only per teammate, even in a workspace that manages its own team list.

`list_teammates` returns each value with its own `integration_id`, so you can see both layers
before touching anything.

## Before any bulk write

1. **`list_smart_fields`** — you write by `smart_field_id` (UUID), never by label. Resolve labels →
   ids once, up front. Note each field's `kind` and `integration_id`.
2. **`list_teammates`** with `auto_paginate` — the current values, and who is affected.
3. **Split the work**: fields you can actually write vs. fields an integration owns. Tell the user
   which ones you cannot change, and where to change them, **before** running the batch — not as an
   apology afterwards.
4. Confirm the whole plan once ("update Job title for these 38 people"), rather than asking per
   person.

## Write safety

`update_teammate` takes the teammate `id` plus `smart_fields: [{ smart_field_id, value }]`. It has
two very different failure modes:

|                                                                                                        | What happens                                                       |
| ------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------ |
| **Loud** — unknown `smart_field_id`, or a value invalid for its kind (`url` and `email` are validated) | 422, **the whole update fails**, nothing changes for that teammate |
| **Silent** — integration-synced fields and `image` fields in the payload                               | Skipped, and the call still returns success                        |

**A 200 does not mean the value changed.** After a bulk run, re-read the affected teammates and
diff against what you sent, then report honestly:

> Updated 38 of 40. 2 skipped — Job title syncs from Google Workspace and is read-only here.

`null` or an empty string clears a value. One call per teammate; there is no batch endpoint.

## People and addresses

- `create_teammate` — primary `email` plus optional initial `smart_fields`.
- `create_email` — an **alias** for an existing teammate (`teammate_id` + `address`). A teammate's
  primary address is set with the teammate and cannot be deleted via `delete_email` (422).
- `update_email` changes an address; `delete_teammate` is a soft delete.
- Signatures are assigned per **email**, not per teammate — a new alias may need its own assignment.
  See the `signature-rollout` skill.

## Changing the field set

- `create_smart_field` — `label` plus `kind` (`text` | `url` | `email` | `phone_number` | `image`).
  Binding it to an integration (`integration_id` + `integration_attribute`) syncs the values
  automatically, which beats maintaining them by hand whenever the data already lives in the
  directory.
- `update_smart_field` — default fields are read-only (422); the computed Email field is not
  addressable (404).
- `delete_smart_field` — **removes that value on every teammate**. Default fields → 422; a field
  still used in a signature → 409, so check where it is used first. This is destructive and
  permanent: confirm explicitly, naming what will be lost.

## Rules of thumb

- An empty value renders as a blank line in every signature using that field. A field empty for most
  of the team is a finding worth raising, not a shrug.
- Verify the outcome where it counts: `preview_signature` for layout, `get_signature_html` for what
  one recipient actually receives.
