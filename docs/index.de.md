# Kimai Jira Sync

Ein [Kimai](https://www.kimai.org/)-Plugin, das Zeiteinträge mit einem Jira-Vorgangsschlüssel
verknüpft und die erfasste Zeit als Worklog nach Jira überträgt – authentifiziert mit dem
**eigenen** persönlichen Zugriffstoken jedes Benutzers (Jira Server / Data Center) oder API-Token
(Jira Cloud). Die Zeiterfassung funktioniert auch dann weiter, wenn Jira nicht erreichbar ist; die
Synchronisierung holt im Hintergrund auf.

![Kimai-Jira-Einstellungsseite: maskierter Token-Hinweis, grüner „Gültig“-Verbindungsstatus sowie
die Schaltflächen „Verbindung testen“ / „Token löschen“.](img/settings-page.png)

## Was es kann

- Fügt jedem Zeiteintrag ein optionales **Jira-Vorgang**-Feld hinzu (z. B. `PROJ-123`), das
  während der Eingabe live geprüft und als anklickbarer Link zum Ticket angezeigt wird.
- Beim Stoppen/Speichern wird ein **Jira-Worklog erstellt oder aktualisiert** – robust gegenüber
  Jira-Ausfällen, mit einem Hintergrund-Abgleich, der alles nachträgt, was inline nicht
  synchronisiert werden konnte.
- Optional die Gegenrichtung: ein aktivierbarer Import holt die **eigenen Jira-Worklogs jedes
  Benutzers als Zeiteinträge** nach Kimai.
- **Routet** importierte Worklogs anhand ihres Jira-Schlüssels ins richtige Kimai-Projekt, kann
  ein Projekt für einen noch nicht beanspruchten Schlüssel **automatisch anlegen** und übernimmt
  ausgewählte Jira-**Benutzerfelder** auf den Eintrag.
- Ein ungültiges Token, eine stockende Synchronisierung oder ein nicht importierbares Feld bleibt
  **nicht unbemerkt** – ein Sitzungsbanner, Eskalations-E-Mails, ein Dashboard-Widget und eine
  wöchentliche Admin-Zusammenfassung.

Jeder Benutzer speichert seine eigenen Zugangsdaten; Token werden **verschlüsselt gespeichert** und
niemals geteilt, protokolliert oder von einer API zurückgegeben.

## Installation

Dies ist ein kostenpflichtiges Plugin, das als Release-ZIP ausgeliefert wird. In das
`var/plugins/`-Verzeichnis Ihrer Kimai-Installation entpacken und den Installer ausführen:

```bash
cd /path/to/kimai
unzip JiraBundle-vX.Y.Z.zip -d var/plugins/   # entpackt nach var/plugins/JiraBundle/
bin/console kimai:bundle:jira:install         # legt die plugin-eigene Tabelle an
bin/console cache:clear
```

Voraussetzungen: **Kimai ≥ 2.21.0**, **PHP ≥ 8.2** und `ext-sodium` (für den verschlüsselten
Token-Speicher).

## Einrichtung

1. Als Administrator die Jira-**Server-URL** und den **Auth-Modus** unter **System →
   Einstellungen → Jira** festlegen.
2. Jeder Benutzer öffnet **Jira-Einstellungen** in seinem Benutzermenü und fügt sein eigenes Token
   ein – eine Schaltfläche **Verbindung testen** meldet ein konkretes Ergebnis (abgelehnte
   Zugangsdaten / DNS / TLS / Zeitüberschreitung).
3. Das **Jira-Vorgang**-Feld an einem Zeiteintrag ausfüllen – das Worklog erscheint, sobald der
   Eintrag mit Endzeit gestoppt oder gespeichert wird.

Einen Cron-Eintrag für den Hintergrund-Abgleich hinzufügen (und, falls der Import aktiviert wird,
für den Importer):

```bash
*/5 * * * * cd /path/to/kimai && bin/console kimai:jira:sync  >> var/log/jira-cron.log 2>&1
0   * * * * cd /path/to/kimai && bin/console kimai:jira:import >> var/log/jira-cron.log 2>&1
```

## Anleitungen

- [Worklog-Sync (Kimai → Jira)](features/worklog-sync.md)
- [Import (Jira → Kimai)](features/importing.md)
- [Projektbezogenes Routing](features/project-routing.md)
- [Automatisches Anlegen (Opt-in)](features/auto-create.md)
- [Benutzerfeld-Übernahme](features/custom-fields.md)
- [Benachrichtigungen & Sichtbarkeit](features/notifications.md)
- [Fehlerbehebung](features/troubleshooting.md)
