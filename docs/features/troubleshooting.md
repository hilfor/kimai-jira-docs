# Troubleshooting

Support requests start with the plugin's own log: `var/log/jira-<env>-<date>.log` (a dedicated
`jira` monolog channel; the token is never logged).

## Jira features stopped working

Sync, import, and issue lookup **all stop together** when the license is inactive or stale — while
Kimai time tracking keeps working. If everything Jira went quiet at once, suspect the license before
anything else.

- **Check the license status.** Look for the admin banner: a grace-window countdown ("*sync keeps
  working for N more day(s)*"), an "inactive" notice, or "no license key configured". Then confirm
  the key under **System → Settings → Jira** (or the `JIRA_LICENSE_KEY` env var, which **wins** over
  the settings field — check both). See [License](../licensing.md).
- **Check the `kimai:jira:sync` cron.** The daily license heartbeat rides this cron and is the only
  thing that resets the offline-staleness clock. If it is not scheduled, a paid, online install
  degrades on its own after ~44 days (30 days offline-staleness + 14 days grace). Schedule it (see
  [Configure → Cron](../configure.md)), or set `JIRA_LICENSE_OFFLINE_GRACE_DAYS=0` for a deliberately
  air-gapped install.
- **Check the log.** On the `jira` channel (`var/log/jira-<env>-<date>.log`), a paused run logs
  `Reconciler skipped: Jira subscription inactive.` (sync) or `Import skipped: Jira subscription
  inactive.` (import). Running the commands by hand prints `Jira subscription inactive — … Enter a
  valid license key under Settings → Jira.` A license-endpoint hiccup logs `Jira license status ping
  failed; failing open.` at info level and is harmless — it never disables a working instance.

Once a valid key is active, run `bin/console kimai:jira:sync` — the heartbeat re-activates features
and any queued worklogs drain.

## Nothing syncs to Jira

- **No token / token invalid for that customer** — open the per-customer token overview
  (**Jira settings** → the customer's row, or `/jira/settings/{user}`) and check that customer's
  status (`valid` / `invalid`), plus the dashboard widget's per-customer breakdown. A `401` pauses
  that user's Jira calls for that customer until the token is updated.
- **The project has no customer, or the customer has no Jira configured** — sync routes by the
  timesheet's customer (timesheet → project → customer). An entry whose project has no customer, or
  whose customer has no `jira_server_url` set, is never synced.
- **Entry has no end time or no issue key** — a worklog is only created once both exist.
- **`sync_mode = manual`** — nothing syncs inline; run `kimai:jira:sync --status=pending`.
- **Jira unreachable** — entries stay `pending`; the reconciler drains them. Check the circuit
  breaker isn't open (repeated connection failures pause inline attempts for a few minutes).

## The importer creates nothing

- No customer has `jira_import_enabled` on, or no target is resolvable — with no per-project
  [routing](project-routing.md), no [auto-create](auto-create.md), and an unset/deleted default
  project or activity **on that customer**, the run reports "not configured" and exits.
- Run `bin/console kimai:jira:import --dry-run` to see what it *would* do, per user and per issue.

## Imported time lands in the wrong project

- Check which project [claims](project-routing.md) that Jira key (its `Jira project key(s)`
  field). A **duplicate claim** (two projects, same key) resolves to the lower project id and is
  logged — fix the duplicate.
- An unclaimed key uses the customer's default import target (or auto-creates under that customer,
  if enabled).

## A custom field isn't imported

- Only [scalar values](custom-fields.md) (text/number/yes-no) are copied. Dropdowns, users, and
  multi-value fields are skipped — check the dashboard widget / admin banner / digest, which name
  the exact field and type.
- The Kimai side must be a real timesheet custom field (System → Customfields) with a lowercase
  name not starting with `jira_`.

## A Jira project or field is missing from a dropdown

- The **project-key** and **custom-field** pickers read their options from Jira, but the answer is
  **cached per user for ~10 minutes** to keep the forms fast. A project, field, or permission you
  just changed in Jira can take up to that long to appear. Wait it out, or run `bin/console
  cache:clear` to drop the cached lookups immediately.
- If it never appears, it's not the cache — the token can't see it: confirm the account has
  permission on that Jira project, then re-open the form.

## Emails don't arrive

- A working `MAILER_DSN` is required, and `ext-xsl` (Kimai core needs it to render any mail).
- Set `framework.router.default_uri` so links in cron-sent mail point at your real domain, not
  `localhost`. Failures are logged on the `jira` channel and never crash the run.

See also: [notifications](notifications.md) · the [overview](../index.md).
