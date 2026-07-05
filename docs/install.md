# Install

This is a paid plugin distributed as a release ZIP. Extract it into your Kimai's `var/plugins/`
directory and run the installer:

```bash
cd /path/to/kimai
unzip JiraBundle-vX.Y.Z.zip -d var/plugins/   # extracts to var/plugins/JiraBundle/
bin/console kimai:bundle:jira:install         # creates the plugin's own table
bin/console cache:clear
```

`kimai:plugins` should now list **JiraBundle**.

## Requirements

- **Kimai** ≥ 2.21.0
- **PHP** ≥ 8.2
- **ext-sodium** — for the encrypted token vault (Kimai's documented PHP setup normally has it).
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

Next: [Configure](configure.md).
