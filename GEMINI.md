# Scribe Mail

Scribe manages a company's email signatures: templates that render for every teammate, the
rules that decide who gets which one, banner campaigns beneath the signature, and the views,
clicks, and conversions they drive. This extension connects you to the user's Scribe workspace
through the `scribe` MCP server (`https://mcp.scribe-mail.com/mcp`, 60+ tools, reads and writes
split). Sign-in is OAuth via WorkOS AuthKit: the first tool call opens the browser, no API key
is pasted anywhere, and you act as the signed-in user with their permissions. Run
`/mcp auth scribe` if a call comes back unauthorized.

## Skills

Seven skills carry the workflows an agent otherwise gets wrong. Load the matching one before
acting.

- **signature-authoring**: Create or edit a Scribe email-signature template: the draft → preview
  → publish → install loop, the block-tree content format, smart fields, and reply/internal
  variants. Use whenever the user wants to build, restyle, fix, or publish a signature in Scribe.
- **signature-rollout**: Decide and apply who gets which Scribe signature (manual recipients,
  smart-field condition rules, org-unit/group targeting, and future-email auto-assignment), then
  verify the rollout landed. Use when assigning, retargeting, or auditing signature coverage.
- **signature-troubleshooting**: Diagnose why a Scribe signature is missing, stale, or wrong in
  someone's mail client by walking installation status, integration health, recipient sources,
  and publish state in the order that finds the cause fastest. Use for "my signature isn't
  showing up / is out of date / looks wrong".
- **marketing-campaigns**: Run a Scribe signature marketing campaign: build the banner content,
  target it, schedule/pause/resume it, and measure it. Use when the user wants to promote an
  event, launch, or offer through the banner under everyone's email signature.
- **signature-analytics**: Answer questions about Scribe signature performance (views, clicks,
  clickthrough rate, conversion events, and identified visitors), choosing timeseries vs
  breakdown correctly and reporting the numbers honestly. Use for any "how is our signature
  performing / who clicks / what did it drive" question.
- **workspace-audit**: Sweep a Scribe workspace for problems in one pass (teammates with no
  signature, failed installations, erroring integrations, unpublished drafts, expired campaigns)
  and report what to fix first. Use for "audit our signatures", "who's missing a signature", or
  a health check before a rollout.
- **teammate-data**: Bulk-manage the people behind the signatures in Scribe (smart-field values,
  adding or removing teammates and alias emails, and the field definitions themselves),
  including which writes silently do nothing because an integration owns the field. Use for
  "update everyone's job title / phone / office address", onboarding or offboarding, or adding
  a new merge field.

## Rules that hold across every skill

- Writing a template never changes what people send. Only `publish_signature` promotes the
  draft to the live copy. Preview before you publish, and say which copy you changed.
- Assignment adds up from four independent sources. Read `list_signature_recipients` before
  "fixing" a recipient, so you do not undo a rule you did not look at.
- Analytics tools are read-only. Report the numbers the tools return, and name the period and
  the breakdown you used.
- A write that returns success can still change nothing when an integration owns the field.
  The `teammate-data` skill lists those cases. Check before running a batch.
- Help-center content is available through the server's documentation tools. Prefer it to
  guessing how a Scribe feature behaves.
