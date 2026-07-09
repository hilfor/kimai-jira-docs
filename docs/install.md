# Install

This is a paid plugin distributed as a release ZIP. Extract it into your Kimai's `var/plugins/`
directory and run the installer:

```bash
cd /path/to/kimai
unzip JiraBundle-vX.Y.Z.zip -d var/plugins/   # extracts to var/plugins/JiraBundle/
bin/console cache:clear
bin/console kimai:bundle:jira:install         # creates the plugin's own table
bin/console cache:clear
```

`kimai:plugins` should now list **JiraBundle**.

## Enter your license key

JiraBundle is a paid plugin — the Jira features stay off until a valid key is set. Paste the key you
received on purchase under **System → Settings → Jira**, or set the `JIRA_LICENSE_KEY` environment
variable (which takes precedence). See [License](licensing.md) for the details, the grace window,
and the `kimai:jira:sync` heartbeat.

## Requirements

- **Kimai** ≥ 2.21.0
- **PHP** ≥ 8.2
- **ext-sodium** — for the encrypted token vault (Kimai's documented PHP setup normally has it).
- **ext-xsl** — required by Kimai core itself, and needed to render any email; without it the
  notification mails silently fail to send.
- A working `MAILER_DSN` and a cron entry — see [Configure](configure.md) — for the notification
  emails and the background reconciler/importer.

Run these from your Kimai root to check all three before installing:

```bash
bin/console --version                 # Kimai version — must be ≥ 2.21.0
php -v | head -n1                     # PHP version — must be ≥ 8.2
php -m | grep -qi sodium && echo "ext-sodium: OK" || echo "ext-sodium: MISSING — install php-sodium"
```

Or as one pass/fail block:

```bash
php -r 'exit(version_compare(PHP_VERSION,"8.2.0",">=")?0:1);' \
  && echo "PHP $(php -r 'echo PHP_VERSION;'): OK" || echo "PHP too old (need ≥ 8.2)"
php -r 'exit(extension_loaded("sodium")?0:1);' \
  && echo "ext-sodium: OK" || echo "ext-sodium: MISSING"
```

`ext-sodium` is a hard requirement — without it the plugin cannot store tokens and the installer
will refuse to run.

## Update

Drop a newer ZIP over the same `var/plugins/JiraBundle` directory, then re-run the installer (it
applies any new migrations) and clear the cache:

```bash
bin/console kimai:bundle:jira:install
bin/console cache:clear
```

The plugin's migrations create and manage its **own** `kimai2_jira_token` table and seed two of its
own rows into Kimai's shared `kimai2_configuration` table — `jira.token_key` (the token-encryption
key) and `jira.license_state` (the license-clock anchor). Both are seeded only when absent, so
re-running the installer on an existing install is safe and never overwrites either value.

!!! warning "Upgrading from a global-config release is a hard cutover"
    This release moves Jira configuration from the old **System → Settings → Jira** page onto each
    **customer**, and turns the single per-user token into one token **per customer**. On upgrade,
    existing tokens are dropped and the old global `jira.*` settings are removed. After updating you
    must **re-configure Jira on each customer** (server URL, auth mode, import options — see
    [Configure](configure.md)) and every user must **re-enter one token per customer**.

!!! warning "The token-encryption key lives in the database — back it up as one unit"
    Stored Jira tokens are encrypted with a key that is generated once at install and kept in your
    Kimai database (the `jira.token_key` row in `kimai2_configuration`). Because the key travels with
    the database, a normal database backup or migration carries it too — restore the database and the
    stored tokens decrypt again. Rotating Kimai's `APP_SECRET` **no longer** affects stored tokens.
    The one thing that orphans tokens is losing that database row: if you restore application files
    onto an **empty** or different database, or wipe the row, every stored token becomes
    undecryptable and each user must re-enter theirs.

!!! note "Advanced: pinning the key with `JIRA_TOKEN_KEY`"
    You normally never touch this. `JIRA_TOKEN_KEY` is an **override** — when set, the vault uses it
    instead of the database-stored key, for deployments that would rather manage the key alongside
    their other secrets. On an install that **already has stored tokens**, do not simply set a fresh
    `JIRA_TOKEN_KEY`: those tokens were encrypted under the database key, and pinning a different key
    orphans them all. To move to a pinned key safely, copy the existing value out of the
    `jira.token_key` row and set `JIRA_TOKEN_KEY` to exactly that, then verify a user can still load
    their saved token before removing the row. Setting `JIRA_TOKEN_KEY` from scratch is only for a
    brand-new install with no tokens yet.

Next: [Configure](configure.md).
