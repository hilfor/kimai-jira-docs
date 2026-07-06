# Deinstallation

Das Plugin bringt einen Deinstallations-Befehl mit, der den Installer rückgängig macht. Er führt
die plugin-eigenen Migrationen wieder *nach unten* aus – entfernt also die plugin-eigene Tabelle
`kimai2_jira_token` (und deren Tracking-Tabelle `bundle_migration_jira`) – und rührt standardmäßig
keine Kimai-Kerntabelle an.

```bash
cd /path/to/kimai
bin/console kimai:bundle:jira:uninstall   # entfernt die plugin-eigene Tabelle
```

Anschließend das Plugin entfernen und den Cache neu aufbauen:

```bash
rm -rf var/plugins/JiraBundle
bin/console cache:clear
```

`kimai:plugins` sollte **JiraBundle** nun nicht mehr auflisten.

## Cron-Einträge entfernen

Wenn Sie den Hintergrund-Reconciler und/oder den Importer eingerichtet haben (siehe
[Einrichtung](configure.md)), entfernen Sie deren beide Cron-Einträge – sie rufen Befehle auf, die
nach der Deinstallation nicht mehr existieren:

- `kimai:jira:sync`
- `kimai:jira:import`

## Was bleibt erhalten und was `--purge` entfernt

Standardmäßig behält die Deinstallation die `jira_*`-Metadaten, die das Plugin an Ihren
**Kunden**, **Projekten** und **Zeiterfassungen** gespeichert hat. Das ist beabsichtigt:

- Der **Jira-Vorgangsschlüssel** an jeder historischen Zeiterfassung bleibt erhalten, sodass Ihre
  Zeiteinträge unverändert bleiben und weiterhin erkennbar ist, zu welchem Ticket sie gehörten.
- Eine spätere **Neuinstallation** knüpft genau dort an, wo Sie aufgehört haben – die
  kundenspezifischen Einstellungen und das projektbezogene Routing sind weiterhin vorhanden.

Diese verbleibenden Metadaten sind nach der Deinstallation wirkungslos: Kimai besitzt keine
Formulare oder Spalten, die sie auslesen, sie werden also schlicht ignoriert.

Um diese Daten ebenfalls zu entfernen, ergänzen Sie `--purge`:

```bash
bin/console kimai:bundle:jira:uninstall --purge
```

`--purge` ist **unwiderruflich**. Es löscht jedes `jira_*`-Metafeld aus den Meta-Tabellen von
Kunden, Projekten und Zeiterfassungen sowie jede veraltete instanzweite `jira.*`-Konfiguration. Sie
werden zuvor um Bestätigung gebeten; mit `--force` überspringen Sie die Rückfrage im
nicht-interaktiven Betrieb (z. B. aus einem Skript):

```bash
bin/console kimai:bundle:jira:uninstall --purge --force
```

!!! warning "Benutzerfelder bleiben auch mit `--purge` erhalten"
    Der Importer kopiert ausgewählte Jira-**Benutzerfelder** unter den **von Ihnen gewählten
    Namen** (z. B. `cost_centre`) auf die Zeiterfassungen, nicht unter einem `jira_`-Präfix.
    `--purge` entfernt ausschließlich `jira_*`-Schlüssel, diese kopierten Werte bleiben daher
    unangetastet – sie sind von manuell eingegebenen Daten nicht zu unterscheiden. Entfernen Sie
    sie bei Bedarf selbst.

## Reihenfolge

1. `kimai:bundle:jira:uninstall` ausführen (mit `--purge`, wenn auch die Daten weg sollen).
2. Die Cron-Einträge `kimai:jira:sync` und `kimai:jira:import` entfernen.
3. `rm -rf var/plugins/JiraBundle`.
4. `bin/console cache:clear`.
