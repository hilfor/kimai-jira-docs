# Configure

## First steps

1. As an admin, set the Jira **server URL** and **auth mode** under **System → Settings → Jira**.
2. Each user opens **Jira settings** from their user menu and pastes their own token — a **Test
   connection** button reports a specific result (rejected credentials / DNS / TLS / timeout).
3. Fill the **Jira issue** field on a timesheet — the worklog appears once the entry is stopped or
   saved with an end time.

![The per-user Jira settings page: a masked token hint, a green "Valid" status, and Test
connection / Delete token buttons.](img/settings-page.png)
*Every user manages their **own** token here — it is encrypted at rest and never shown back.*

## Authentication

- **Jira Server / Data Center** — `auth_mode = bearer`; each user pastes a **personal access
  token** (Jira profile → Personal Access Tokens).
- **Jira Cloud** — `auth_mode = basic`; each user pastes an **API token**
  ([id.atlassian.com](https://id.atlassian.com/manage-profile/security/api-tokens)) and sets their
  Jira account email under Kimai preferences (section "Jira").

The server URL is a security boundary — every user's token is sent to whatever host it names — so
it is **validated when you save it**: it must use `https://` (tokens are never sent in clear text)
and must not point at a private, loopback, or link-local address. Changing it re-validates every
stored token on the next daily heartbeat, since a token proven against the old host says nothing
about a new one.

## Cron

The reconciler and (optional) importer run from cron — add these as the user Kimai's own cron runs
as:

```bash
# Reconciler: drains anything that couldn't sync inline (every 5 minutes)
*/5 * * * * cd /path/to/kimai && bin/console kimai:jira:sync   >> var/log/jira-cron.log 2>&1

# Importer: pulls each user's own Jira worklogs into Kimai (opt-in; e.g. hourly)
0   * * * * cd /path/to/kimai && bin/console kimai:jira:import  >> var/log/jira-cron.log 2>&1

# Admin weekly digest (Monday mornings)
0 8 * * 1   cd /path/to/kimai && bin/console kimai:jira:sync --digest >> var/log/jira-cron.log 2>&1
```

For cron-sent emails to link back to your instance, set `framework.router.default_uri` in your
Kimai config so links resolve to your real domain instead of `localhost`.

## Instance settings reference

The full set of instance-wide settings lives under **System → Settings → Jira**. What each one
does is covered in the guides:

- Import target, per-project routing, auto-create — [Importing](features/importing.md),
  [Per-project routing](features/project-routing.md), [Auto-create](features/auto-create.md).
- Custom-field mapping — [Custom-field passthrough](features/custom-fields.md).
- Notifications — [Notifications & visibility](features/notifications.md).
