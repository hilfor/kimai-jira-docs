# Worklog sync (Kimai → Jira)

The core feature: tag a timesheet with a Jira issue and push the tracked time to Jira as a
worklog.

## The Jira issue field

Every timesheet entry (create/edit form and the start/stop toolbar) has an optional **Jira
issue** field (e.g. `PROJ-123`, validated `^[A-Z][A-Z0-9_]*-\d+$`). Leave it empty to keep the
entry out of Jira. As a timesheet column it renders as a **clickable link** to the ticket, and
the key is **live-validated** against Jira as you type (showing the issue summary or a "not found"
hint) — advisory only, it never blocks saving.

## When a worklog is created

A worklog is created/updated once the entry has **both** an end time and an issue key **and** the
owning user has a token. A running timer is never synced mid-flight. Editing the key after a
worklog exists deletes the old worklog and creates a new one under the new key. The worklog
comment is the timesheet description (toggle with `jira.sync_comment`).

## Kimai is the source of truth

Saving/stopping never waits on Jira — the HTTP call is deferred after the response, with tight
timeouts. Transient failures (network, `5xx`, `429`) are retried with backoff; permanent ones
(`401`/`403`/`404`/`400`) surface in the sync-status column and, for a `401`, pause that user's
Jira calls until the token is fixed. A background reconciler (`kimai:jira:sync`, from cron) drains
anything that could not sync inline.

## Setup

See the main [README](../index.md) for the instance settings (server URL, auth mode, sync
mode), the per-user token page, and the cron entry for `kimai:jira:sync`.

See also: [importing](importing.md) · [notifications](notifications.md) ·
[troubleshooting](troubleshooting.md).
