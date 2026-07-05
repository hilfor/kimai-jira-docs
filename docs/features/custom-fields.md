# Custom-field passthrough

Copy selected Jira **custom fields** (a cost centre, a client reference, …) onto imported
timesheets, so a value that already lives in Jira lands in Kimai instead of being retyped.

## Configuring the mapping

Under **System → Settings → Jira**, set `jira.import_fields` — one mapping per line:

```text
customfield_10012 = cost_centre
duedate = due_date
```

- **Left** = the Jira field id (`customfield_<n>`, or a system field like `duedate`). You rarely
  need to look one up by hand: with a valid Jira token, an **Insert a Jira field…** helper below
  the mapping box lists your Jira fields (id + human name) and appends the `customfield_… = ` line
  for you. You can also find a field's id in Jira under *Settings → Issues → Custom fields* or via
  `GET /rest/api/2/field`.
- **Right** = the Kimai **timesheet custom field** name to copy it into. It must be lowercase
  (letters, digits, underscore) and cannot start with `jira_` (reserved). Create the matching
  Kimai timesheet custom field first (System → Customfields) so there is somewhere to store it.
- `#` comments and blank/garbage lines are ignored; a duplicate Jira id — the last line wins.

## Scalars only — and skipped fields are surfaced

Only **simple values** map cleanly: text, numbers, and yes/no (stored as `1`/`0`). Dates arrive
as text and are copied as-is.

Richer Jira field types — single-select (`{value: …}`), user (`{displayName: …}`), multi-select
(a list) — are **not** imported. Instead of failing or guessing, the importer **skips them and
tells you**, on every channel:

- the **dashboard widget** ("2 Jira fields could not be imported"),
- an **admin banner** on the next page load,
- the **weekly digest** email,
- and the `kimai:jira:import` command output (`--dry-run` predicts it too).

Each warning names the exact field and its type, so if you keep hitting one (a select-typed cost
centre is common), you know precisely what to ask for next. Fixing or removing a mapping clears
its warning automatically on the next run.

See also: [importing](importing.md) · [notifications](notifications.md).
