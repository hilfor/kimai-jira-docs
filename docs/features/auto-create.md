# Opt-in auto-create

When you track time against many Jira tickets, pre-creating a Kimai project for each is toil.
Auto-create lets **structure emerge from real work**: a Jira key that no project claims yet gets a
Kimai project created for it on import.

## Enabling it

On the **customer's edit form** (Customers → edit a customer → Jira):

- `jira_autocreate_enabled` = **on** (default **off** — it creates data).

There is no separate customer setting to choose: auto-created projects are filed under **the
customer whose config triggered the import** (that customer's own Jira). You also still need that
customer's `jira_import_activity` — auto-create makes *projects*, not activities, so imported
entries use the customer's import activity until you set a per-project override.

## What happens on import

For a Jira key that no existing project [claims](project-routing.md):

1. A Kimai project is created, **named after the Jira project** (from Jira's project name; the
   bare key if the name can't be read), filed under **this customer** (the one being imported).
2. The project already **claims that key** (its `Jira project key(s)` field is pre-filled), so
   from the next run on it is an ordinary routed project you can rename, re-parent, or set rates
   on.
3. The imported worklogs are filed under it (with the customer's import activity).

## Guarantees

- **Opt-in only** — otherwise the importer falls back to the customer's default target and logs
  why.
- **Idempotent** — one project per Jira key per run, and re-runs find the existing claim instead
  of creating duplicates.
- **Never destructive / never crashes** — a failed creation falls back to the customer's default
  and is logged; the run continues.
- **`--dry-run`** reports the projects it *would* create without writing anything.
- **No activities** are auto-created.

See also: [per-project routing](project-routing.md) · [importing](importing.md).
