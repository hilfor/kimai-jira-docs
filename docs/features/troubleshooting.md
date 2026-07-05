# Troubleshooting

Support requests start with the plugin's own log: `var/log/jira-<env>-<date>.log` (a dedicated
`jira` monolog channel; the token is never logged).

## Nothing syncs to Jira

- **No token / token invalid** — check the per-user Jira settings page (status `valid` /
  `invalid`) and the dashboard widget. A `401` pauses that user's Jira calls until the token is
  updated.
- **Entry has no end time or no issue key** — a worklog is only created once both exist.
- **`sync_mode = manual`** — nothing syncs inline; run `kimai:jira:sync --status=pending`.
- **Jira unreachable** — entries stay `pending`; the reconciler drains them. Check the circuit
  breaker isn't open (repeated connection failures pause inline attempts for a few minutes).

## The importer creates nothing

- `jira.import_enabled` is off, or no target is resolvable — with no per-project
  [routing](project-routing.md), no [auto-create](auto-create.md), and an unset/deleted default
  project or activity, the run reports "not configured" and exits.
- Run `bin/console kimai:jira:import --dry-run` to see what it *would* do, per user and per issue.

## Imported time lands in the wrong project

- Check which project [claims](project-routing.md) that Jira key (its `Jira project key(s)`
  field). A **duplicate claim** (two projects, same key) resolves to the lower project id and is
  logged — fix the duplicate.
- An unclaimed key uses the global default (or auto-creates, if enabled).

## A custom field isn't imported

- Only [scalar values](custom-fields.md) (text/number/yes-no) are copied. Dropdowns, users, and
  multi-value fields are skipped — check the dashboard widget / admin banner / digest, which name
  the exact field and type.
- The Kimai side must be a real timesheet custom field (System → Customfields) with a lowercase
  name not starting with `jira_`.

## Emails don't arrive

- A working `MAILER_DSN` is required, and `ext-xsl` (Kimai core needs it to render any mail).
- Set `framework.router.default_uri` so links in cron-sent mail point at your real domain, not
  `localhost`. Failures are logged on the `jira` channel and never crash the run.

See also: [notifications](notifications.md) · main [README](../index.md).
