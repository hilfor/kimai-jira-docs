# Uninstall

The plugin ships an uninstall command that reverses the installer. It runs the plugin's own
migrations back *down* — dropping the plugin's own `kimai2_jira_token` table (and its
`bundle_migration_jira` tracking table) — and never touches a core Kimai table by default.

```bash
cd /path/to/kimai
bin/console kimai:bundle:jira:uninstall   # drops the plugin's own table
```

Then remove the plugin and rebuild the cache:

```bash
rm -rf var/plugins/JiraBundle
bin/console cache:clear
```

`kimai:plugins` should no longer list **JiraBundle**.

## Remove the cron entries

If you set up the background reconciler and/or importer (see [Configure](configure.md)), remove
their two cron entries — they call commands that no longer exist once the plugin is gone:

- `kimai:jira:sync`
- `kimai:jira:import`

## What is kept, and what `--purge` removes

By default the uninstall keeps the `jira_*` meta the plugin stored on your **customers**,
**projects** and **timesheets**. That is deliberate:

- The **Jira issue key** on each historical timesheet is preserved, so your time records stay
  intact and still show which ticket they belonged to.
- A later **reinstall** picks up exactly where you left off — per-customer settings and per-project
  routing are still there.

This leftover meta is inert once the plugin is gone: Kimai has no forms or columns that read it, so
it is simply ignored.

To remove that data as well, add `--purge`:

```bash
bin/console kimai:bundle:jira:uninstall --purge
```

`--purge` is **irreversible**. It deletes every `jira_*` meta field from the customer, project and
timesheet meta tables and any legacy instance-wide `jira.*` configuration. You are asked to confirm
first; pass `--force` to skip the prompt when running non-interactively (e.g. from a script):

```bash
bin/console kimai:bundle:jira:uninstall --purge --force
```

!!! warning "Custom fields are kept even with `--purge`"
    The importer copies selected Jira **custom fields** onto timesheets under the **names you
    chose** (e.g. `cost_centre`), not under a `jira_` prefix. `--purge` only ever removes
    `jira_*` keys, so those copied values are left untouched — they are indistinguishable from
    data you might have entered by hand. Remove them yourself if you no longer want them.

## Order of operations

1. Run `kimai:bundle:jira:uninstall` (add `--purge` if you also want the data gone).
2. Remove the `kimai:jira:sync` and `kimai:jira:import` cron entries.
3. `rm -rf var/plugins/JiraBundle`.
4. `bin/console cache:clear`.
