# Per-project routing

By default the importer files every imported worklog under one project
(`jira.import_project`). That is wrong when your Jira instance has **many projects** — you want
`PROJ-123` under one Kimai project and `OPS-9` under another. Per-project routing does that.

![The Jira project-key and import-activity fields on a Kimai project's edit form.](../img/project-jira-keys.png)

## How it works

Each Kimai project **claims** the Jira project key(s) it owns. Open a project's normal edit form
(Administration → Projects → *edit*) and fill in:

- **Jira project key(s)** — one or more uppercase Jira keys, comma-separated (e.g. `PROJ` or
  `PROJ, OPS`). Any imported worklog on an issue like `PROJ-123` is filed under this project.
- **Jira import activity** *(optional)* — the activity imported entries for this project use,
  overriding the global `jira.import_activity`. Leave empty to use the global default.

The mapping lives **on the project**, so it is created and deleted with the project — there is no
separate table or screen to keep in sync.

!!! tip "Autocomplete"
    When you have a valid Jira token (from your **Jira settings**), the **Jira project key(s)**
    field suggests your Jira projects as you type (`KEY — Name`), so you don't have to remember
    the exact keys. Without a token it stays a plain text field — the feature is pure convenience.

## Resolution order

For each imported issue key, the importer picks the target in this order:

1. **A project that claims the key** → that project (+ its activity override, or the global
   activity).
2. **No project claims it** → [auto-create](auto-create.md) a project (if enabled), otherwise the
   global default `jira.import_project` / `jira.import_activity`.
3. **Nothing resolvable** (unclaimed key, no default) → the issue is skipped and counted in the
   run summary, never silently dropped.

**Backward compatible:** if no project claims any key, every worklog uses the global default —
exactly the single-target behaviour from before this feature existed.

## Notes

- Keys are matched case-insensitively and must look like a Jira project key
  (`^[A-Z][A-Z0-9_]*$`); malformed entries are ignored.
- **Duplicate claim** — if two projects claim the same key, the lower project id wins and the
  conflict is logged and counted, so you can spot and fix it.
- A key claimed by a project that was since deleted/hidden falls back to the global default for
  that run.

See also: [importing](importing.md) · [auto-create](auto-create.md).
