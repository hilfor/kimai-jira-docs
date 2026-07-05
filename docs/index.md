# Kimai Jira Sync

A [Kimai](https://www.kimai.org/) plugin that tags timesheet entries with a Jira issue key and
pushes tracked time to Jira as worklogs — authenticated with each user's **own** personal access
token (Jira Server / Data Center) or API token (Jira Cloud). Time tracking keeps working even when
Jira is unreachable; the sync catches up in the background.

![Kimai Jira settings page: a masked token hint, a green "Valid" connection status, and Test
connection / Delete token buttons.](img/settings-page.png)

## What it does

- Adds an optional **Jira issue** field (e.g. `PROJ-123`) to every timesheet, live-validated as
  you type and shown as a clickable link to the ticket.
- On stop/save, **creates or updates a Jira worklog** on that issue — resilient to Jira downtime,
  with a background reconciler that drains anything that couldn't sync inline.
- Optionally the reverse direction: an opt-in importer pulls each user's **own Jira worklogs into
  Kimai** as timesheets.
- **Routes** imported worklogs to the right Kimai project by their Jira key, can **auto-create** a
  project for a key nothing claims yet, and copies selected Jira **custom fields** onto the entry.
- A dead token, a stalled sync, or a field it couldn't import is **hard to miss** — a session
  banner, escalation emails, a dashboard widget, and a weekly admin digest.

Each user stores their own credential; tokens are **encrypted at rest** and never shared, logged,
or returned by any API.

## Install

This is a paid plugin distributed as a release ZIP. Extract it into your Kimai's `var/plugins/`
and run the installer:

```bash
cd /path/to/kimai
unzip JiraBundle-vX.Y.Z.zip -d var/plugins/   # extracts to var/plugins/JiraBundle/
bin/console kimai:bundle:jira:install         # creates the plugin's own table
bin/console cache:clear
```

Requires **Kimai ≥ 2.21.0**, **PHP ≥ 8.2**, and `ext-sodium` (for the encrypted token vault).

## Configure

1. As an admin, set the Jira **server URL** and **auth mode** under **System → Settings → Jira**.
2. Each user opens **Jira settings** from their user menu and pastes their own token — a **Test
   connection** button reports a specific result (rejected credentials / DNS / TLS / timeout).
3. Fill the **Jira issue** field on a timesheet — the worklog appears once the entry is stopped or
   saved with an end time.

Add a cron entry for the background reconciler (and, if you enable importing, the importer):

```bash
*/5 * * * * cd /path/to/kimai && bin/console kimai:jira:sync  >> var/log/jira-cron.log 2>&1
0   * * * * cd /path/to/kimai && bin/console kimai:jira:import >> var/log/jira-cron.log 2>&1
```

## Guides

- [Worklog sync (Kimai → Jira)](features/worklog-sync.md)
- [Importing (Jira → Kimai)](features/importing.md)
- [Per-project routing](features/project-routing.md)
- [Opt-in auto-create](features/auto-create.md)
- [Custom-field passthrough](features/custom-fields.md)
- [Notifications & visibility](features/notifications.md)
- [Troubleshooting](features/troubleshooting.md)
