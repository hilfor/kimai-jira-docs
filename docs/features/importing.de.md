# Import von Jira-Worklogs (Jira → Kimai)

Die Gegenrichtung: Entwickler, die ihre Zeit in Jira buchen, können diese Worklogs als Zeiteinträge
nach Kimai fließen lassen, statt sie doppelt zu erfassen.

## Identitätsmodell – nur eigene Worklogs

Der Importer läuft pro **(Kunde, Benutzer)**-Token: Für jeden Kunden mit aktiviertem Import
verwendet er das **eigene** gespeicherte, verschlüsselte Token jedes Benutzers für diesen Kunden als
seine Identität (JQL `worklogAuthor = currentUser()`). Kein Admin-/Dienst-Token, kein
E-Mail-Abgleich – ein Benutzer importiert nur Worklogs, die er selbst in der Jira dieses Kunden
verfasst hat.

![Die Jira-Einstellungen eines Kunden: Server, Auth, Import, automatisches Anlegen und
Benutzerfelder.](../img/system-settings-jira.png)

## Aktivieren + Ziel

Der Import ist **standardmäßig aus** (er legt Daten an). Er wird **pro Kunde** aktiviert, auf dem
Kunden-Bearbeitungsformular (Kunden → Kunde bearbeiten → Jira):

- `jira_import_enabled` = an
- `jira_import_project` / `jira_import_activity` = das **Standard**-Ziel für diesen Kunden (siehe
  [projektbezogenes Routing](project-routing.md) und [automatisches Anlegen](auto-create.md), um
  verschiedene Jira-Projekte an verschiedene Kimai-Projekte zu senden)
- `jira_import_window_days` = wie weit zurück gesucht wird (Standard 14)

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
