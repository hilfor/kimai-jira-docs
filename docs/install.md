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

The plugin's migrations only ever touch its **own** `kimai2_jira_token` table — they never alter
Kimai's core tables — so re-running the installer on an existing install is safe.

!!! warning "Upgrading from a global-config release is a hard cutover"
    This release moves Jira configuration from the old **System → Settings → Jira** page onto each
    **customer**, and turns the single per-user token into one token **per customer**. On upgrade,
    existing tokens are dropped and the old global `jira.*` settings are removed. After updating you
    must **re-configure Jira on each customer** (server URL, auth mode, import options — see
    [Configure](configure.md)) and every user must **re-enter one token per customer**.

!!! warning "Don't rotate `APP_SECRET` without a dedicated token key"
    Stored Jira tokens are encrypted with a key derived from Kimai's `APP_SECRET` by default. If
    `APP_SECRET` changes, **every stored token becomes undecryptable** and each user must re-enter
    theirs. If your deployment rotates `APP_SECRET`, set a dedicated `JIRA_TOKEN_KEY` environment
    variable first — the vault then derives its key from that, so the two can rotate independently.

Next: [Configure](configure.md).
