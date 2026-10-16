---
name: signature-rollout
description: Decide and apply who gets which Scribe signature (manual recipients, smart-field condition rules, org-unit/group targeting, and future-email auto-assignment), then verify the rollout landed. Use when assigning, retargeting, or auditing signature coverage.
---

# Rolling a signature out to the right people

Assignment in Scribe has **four independent sources**, and they add up rather than override
each other. `list_signature_recipients` returns each recipient with a `sources` array telling
you _why_ they are assigned, plus the current `selection` rule config. Read it before changing
anything — otherwise you will "fix" a person who is matched by a rule you did not look at.

| Source                 | Tools                                                                                                 | Use it for                           |
| ---------------------- | ----------------------------------------------------------------------------------------------------- | ------------------------------------ |
| Manual emails          | `add_signature_recipients` / `remove_signature_recipients` (`signature_id` + `email_ids`)             | Exceptions, pilots, one-off people   |
| Smart-field conditions | `set_signature_recipient_rules` (`conditions`)                                                        | "Everyone whose Department is Sales" |
| Org units / groups     | `add_signature_recipient_entities` / `remove_signature_recipient_entities` (`integration_entity_ids`) | Directory-driven teams               |
| Future emails          | `set_signature_recipient_rules` (`assign_to_all_future_emails`)                                       | The company default signature        |

## Procedure

1. `list_signatures` to get the template's `id`; `list_teammates` / `list_emails` to resolve
   people to `email_id`s (each email row carries its owning `teammate_id`; aliases and primary
   addresses are both listed).
2. For directory targeting, `list_integration_entities` gives the org units and groups
   (`uuid`, `kind`, `name`, `teammates_count`) you can point a signature at.
3. Apply the narrowest change that expresses the intent. Prefer conditions or org units over
   pasting a long list of email ids — those keep working as people join and leave.
4. **`set_signature_recipient_rules` replaces the whole `conditions` tree.** It is an array of
   OR-groups, each an array of ANDed conditions. Read the current tree first and send the full
   intended result; `[]` clears every condition. Manual and org-unit targeting are untouched.
5. Confirm with the user before applying — this changes what goes out on real email.
6. Verify: re-run `list_signature_recipients`, then `list_signature_installations` to confirm
   the signature actually installed (see the `signature-troubleshooting` skill when rows come
   back `failed`).

## Gotchas

- Removing manual recipients does **not** remove people matched by a condition or an org unit.
  If someone still has the signature after a removal, look at their `sources`.
- A workspace whose team list is managed by a Google, Microsoft or Zoho Mail directory integration
  rejects teammate and email create/delete with **409** — the directory is the source of truth.
  Change it there, not in Scribe.
- Assignment is per **email**, not per teammate: a teammate with aliases can have different
  signatures on different addresses.
- Coverage question ("who has no signature?"): list emails, list recipients per signature, and
  diff. The `workspace-audit` skill does this sweep end to end.
