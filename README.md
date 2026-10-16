# Scribe Mail extension for Gemini CLI and Antigravity CLI

Manage your company's email signatures from the terminal. This extension connects Gemini CLI or
Antigravity CLI to your [Scribe](https://www.scribe-mail.com) workspace through the Scribe MCP
server and adds the workflow knowledge to use it well: author and publish signature templates,
roll them out to the right teammates, run signature marketing campaigns, and read engagement
analytics.

## Install

Gemini CLI (reads `gemini-extension.json`):

```bash
gemini extensions install https://github.com/scribemail/gemini-cli-extension
```

Antigravity CLI (reads `plugin.json` and `mcp_config.json` from the same repository):

```bash
agy plugin install https://github.com/scribemail/gemini-cli-extension
```

The first tool call opens the Scribe sign-in in your browser. Authentication is OAuth 2.0 via
WorkOS AuthKit. **No API key is pasted anywhere**, and the session acts as you, in your own
workspace, with your own permissions. In Gemini CLI, run `/mcp auth scribe` if a call comes back
unauthorized.

## What it adds

**The Scribe MCP server** (`https://mcp.scribe-mail.com/mcp`): 60+ tools covering signature
templates and variants, teammates, emails, smart fields, folders, assets, integrations,
recipients and installations, marketing campaigns, analytics, and the Scribe help center.

**A context file** (`GEMINI.md`) that tells the agent how Scribe works before it touches
anything: draft versus live content, how assignment rules stack, and which writes an integration
owns.

**Seven skills** (`skills/*/SKILL.md`) that carry the workflows an agent otherwise gets wrong:

| Skill                       | Covers                                                                                                                                    |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `signature-authoring`       | The draft → preview → publish → install loop, the block-tree content format, smart fields, reply/internal variants                        |
| `signature-rollout`         | Manual recipients vs. condition rules vs. org-unit targeting vs. future-email auto-assign, and how they stack                             |
| `signature-troubleshooting` | Why a signature is missing, stale, or wrong, in the order that finds the cause fastest                                                    |
| `marketing-campaigns`       | Banner content, targeting, the draft → schedule → pause → resume lifecycle                                                                |
| `signature-analytics`       | Views, clicks, clickthrough rate, conversion events, visitors; timeseries vs. breakdown                                                   |
| `workspace-audit`           | A one-pass health sweep: no-signature teammates, failed installs, erroring integrations, unpublished drafts                               |
| `teammate-data`             | Bulk smart-field edits, people and aliases, field definitions, and which writes silently do nothing because an integration owns the field |

## Try it

```
Which teammates are missing an email signature?
Audit our Scribe workspace and tell me what to fix first.
Create a signature template for the sales team and show me a preview.
Why isn't ana@acme.com's signature showing up in Outlook?
Show me signature clickthrough by country for the last 30 days.
Launch a campaign banner for our user conference, running 1–15 October.
```

## Safety

Reads run freely. Every write (creating, publishing, installing, assigning, scheduling,
deleting) changes what goes out on real company email, so the skills preview before they
publish and ask before they write, every time.

## Requirements

- A [Scribe](https://www.scribe-mail.com) workspace, and an owner or admin account in it
  (signature and analytics scopes ride on those roles).
- Gemini CLI with extensions support, or Antigravity CLI. Google replaced Gemini CLI with
  Antigravity CLI for individual accounts on 18 June 2026; Gemini CLI stays available on
  Gemini Code Assist Standard and Enterprise licenses.

## Also available for

- Claude Code and Cowork: [scribemail/claude-plugin](https://github.com/scribemail/claude-plugin)
- Cursor: [scribemail/cursor-plugin](https://github.com/scribemail/cursor-plugin)

## Links

- Documentation: <https://help.scribe-mail.com/en/ai-agents>
- Scribe: <https://www.scribe-mail.com>
- Support: <support@scribe-mail.com>

## License

MIT, see [LICENSE](./LICENSE). The skills and this layout are mirrored from the private
`scribemail/mcp-server` repository; open issues here, fixes land there first.
