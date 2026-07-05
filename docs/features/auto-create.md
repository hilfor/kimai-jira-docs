# Opt-in auto-create

When you track time against many Jira tickets, pre-creating a Kimai project for each is toil.
Auto-create lets **structure emerge from real work**: a Jira key that no project claims yet gets a
Kimai project created for it on import.

## Enabling it

Under **System → Settings → Jira**:

- `jira.autocreate_enabled` = **on** (default **off** — it creates data).
- `jira.autocreate_customer` = the customer new projects are filed under. Auto-create stays off
  until this is set.

You also still need a global `jira.import_activity` — auto-create makes *projects*, not
activities, so imported entries use the global activity until you set a per-project override.

## What happens on import

For a Jira key that no existing project [claims](project-routing.md):

1. A Kimai project is created, **named after the Jira project** (from Jira's project name; the
   bare key if the name can't be read), filed under the configured customer.
2. The project already **claims that key** (its `Jira project key(s)` field is pre-filled), so
   from the next run on it is an ordinary routed project you can rename, re-parent, or set rates
   on.
3. The imported worklogs are filed under it (with the global import activity).

## Guarantees

- **Opt-in only** and needs a customer — otherwise the importer falls back to the global default
  and logs why.
- **Idempotent** — one project per Jira key per run, and re-runs find the existing claim instead
  of creating duplicates.
- **Never destructive / never crashes** — a failed creation falls back to the default and is
  logged; the run continues.
- **`--dry-run`** reports the projects it *would* create without writing anything.
- **No activities** are auto-created.

See also: [per-project routing](project-routing.md) · [importing](importing.md).
