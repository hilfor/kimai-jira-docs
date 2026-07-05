# Import von Jira-Worklogs (Jira → Kimai)

Die Gegenrichtung: Entwickler, die ihre Zeit in Jira buchen, können diese Worklogs als Zeiteinträge
nach Kimai fließen lassen, statt sie doppelt zu erfassen.

## Identitätsmodell – nur eigene Worklogs

Der Importer verwendet das **eigene** gespeicherte, verschlüsselte Token jedes Benutzers als seine
Identität (JQL `worklogAuthor = currentUser()`). Kein Admin-/Dienst-Token, kein E-Mail-Abgleich –
ein Benutzer importiert nur Worklogs, die er selbst verfasst hat.

## Aktivieren + Ziel

Der Import ist **standardmäßig aus** (er legt Daten an). Unter System → Einstellungen → Jira:

- `jira.import_enabled` = an
- `jira.import_project` / `jira.import_activity` = das **Standard**-Ziel (siehe
  [projektbezogenes Routing](project-routing.md) und [automatisches Anlegen](auto-create.md), um
  verschiedene Jira-Projekte an verschiedene Kimai-Projekte zu senden)
- `jira.import_window_days` = wie weit zurück gesucht wird (Standard 14)

Aus einem eigenen Cron-Eintrag ausführen, getrennt vom Abgleich:

```bash
0 * * * * cd /path/to/kimai && bin/console kimai:jira:import >> var/log/jira-cron.log 2>&1
```

## Verhalten

- Findet die im Zeitfenster gebuchten Vorgänge jedes Benutzers, importiert dessen eigene Worklogs
  und speichert am Zeiteintrag den Jira-Schlüssel, die Worklog-ID und `jira_sync_status=synced`.
- **Dedupliziert** anhand bereits gespeicherter Worklog-IDs – erneute Läufe erzeugen nie
  Duplikate, und es wird nie ein Worklog erneut importiert, das die eigene Auswärts-Sync von Kimai
  erzeugt hat.
- **Zeitzonen-korrekt** – der Jira-Zeitpunkt `started` wird exakt in die Zeitzone des Benutzers
  überführt.
- **Routet** anhand des Jira-Schlüssels, legt optional Projekte **automatisch an** und übernimmt
  zugeordnete [Benutzerfelder](custom-fields.md).
- `--dry-run` meldet ohne zu schreiben; `--user=ID` beschränkt auf einen Benutzer; `--days=N`
  überschreibt das Zeitfenster.

## Bekannte Einschränkung (MVP)

Importierte Einträge werden direkt gespeichert (um eine Sync-Rückkopplungsschleife zu vermeiden),
was Kimais Satz-Berechnung umgeht – sie starten daher mit **Null-/Standardsätzen**. Bearbeiten Sie
einen Eintrag oder lassen Sie Ihre üblichen Satzregeln beim nächsten Speichern greifen.

Siehe auch: [Projektbezogenes Routing](project-routing.md) · [Automatisches Anlegen](auto-create.md)
· [Benutzerfelder](custom-fields.md).
