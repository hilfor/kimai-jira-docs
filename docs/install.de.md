# Installation

Dies ist ein kostenpflichtiges Plugin, das als Release-ZIP ausgeliefert wird. Entpacken Sie es in
das `var/plugins/`-Verzeichnis Ihrer Kimai-Installation und führen Sie den Installer aus:

```bash
cd /path/to/kimai
unzip JiraBundle-vX.Y.Z.zip -d var/plugins/   # entpackt nach var/plugins/JiraBundle/
bin/console kimai:bundle:jira:install         # legt die plugin-eigene Tabelle an
bin/console cache:clear
```

`kimai:plugins` sollte nun **JiraBundle** auflisten.

## Voraussetzungen

- **Kimai** ≥ 2.21.0
- **PHP** ≥ 8.2
- **ext-sodium** – für den verschlüsselten Token-Speicher (Kimais dokumentierte PHP-Einrichtung
  hat es normalerweise bereits).
- Ein funktionierendes `MAILER_DSN` und ein Cron-Eintrag – siehe [Einrichtung](configure.md) – für
  die Benachrichtigungs-E-Mails und den Hintergrund-Abgleich/Importer.

Führen Sie diese Befehle im Kimai-Stammverzeichnis aus, um alle drei vor der Installation zu
prüfen:

```bash
bin/console --version                 # Kimai-Version – muss ≥ 2.21.0 sein
php -v | head -n1                     # PHP-Version – muss ≥ 8.2 sein
php -m | grep -qi sodium && echo "ext-sodium: OK" || echo "ext-sodium: FEHLT — php-sodium installieren"
```

Oder als Pass/Fail-Block:

```bash
php -r 'exit(version_compare(PHP_VERSION,"8.2.0",">=")?0:1);' \
  && echo "PHP $(php -r 'echo PHP_VERSION;'): OK" || echo "PHP zu alt (≥ 8.2 nötig)"
php -r 'exit(extension_loaded("sodium")?0:1);' \
  && echo "ext-sodium: OK" || echo "ext-sodium: FEHLT"
```

`ext-sodium` ist zwingend erforderlich – ohne die Erweiterung kann das Plugin keine Token
speichern und der Installer verweigert die Ausführung.

## Aktualisieren

Legen Sie ein neueres ZIP über dasselbe Verzeichnis `var/plugins/JiraBundle`, führen Sie dann den
Installer erneut aus (er wendet neue Migrationen an) und leeren Sie den Cache:

```bash
bin/console kimai:bundle:jira:install
bin/console cache:clear
```

Weiter: [Einrichtung](configure.md).
