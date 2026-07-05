# Notifications & visibility

A dead token, a stalled sync, or a field the importer could not read must never fail silently —
"weeks of missing worklogs discovered at invoicing time" is exactly what this layer prevents. All
surfaces are driven from the cron jobs and can never crash a page load or the reconciler (a broken
mail transport is logged and swallowed).

## Token / sync problems

- **In-app banner** — while your token is invalid, the next page load shows a warning once per
  session (it reappears next session, but does not nag on every page).
- **Escalation email with backoff** — on a token going valid → invalid, an email goes out
  immediately, again after 7 days, then monthly while unresolved. The same schedule fires for a
  user whose oldest unsynced entry is more than 7 days old, even with a valid token.
- **Dashboard widget** — a "Jira sync" card with token status, unsynced count, and the age of the
  oldest unsynced entry.
- **Admin weekly digest** — `bin/console kimai:jira:sync --digest` emails every super-admin a
  one-line summary. Run it from its own weekly cron entry.

## Import warnings (custom fields)

When the importer [skips a custom field](custom-fields.md) it could not import, the same idea
applies — the skip is **seen, not grepped**:

- a line on the **dashboard widget** ("N Jira fields could not be imported"),
- an **admin banner** (once per session, for users who can open System settings), linking to
  System → Settings → Jira,
- a line in the **weekly digest**,
- and the `kimai:jira:import` command output.

The warning set is rewritten every run, so fixing a mapping clears it everywhere on the next run.

## Requirements

Escalation/digest emails need a working `MAILER_DSN` and an absolute link back to Kimai — set
`framework.router.default_uri` (or the router request context) so links in cron-sent mail resolve
to your real domain. See the [README](../index.md) for details.

See also: [worklog sync](worklog-sync.md) · [custom fields](custom-fields.md).
