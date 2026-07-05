# Import von Jira-Worklogs (Jira â Kimai)

Die Gegenrichtung: Entwickler, die ihre Zeit in Jira buchen, kÃ¶nnen diese Worklogs als ZeiteintrÃ¤ge
nach Kimai flieÃen lassen, statt sie doppelt zu erfassen.

## IdentitÃ¤tsmodell â nur eigene Worklogs

Der Importer verwendet das **eigene** gespeicherte, verschlÃ¼sselte Token jedes Benutzers als seine
IdentitÃ¤t (JQL `worklogAuthor = currentUser()`). Kein Admin-/Dienst-Token, kein E-Mail-Abgleich â
ein Benutzer importiert nur Worklogs, die er selbst verfasst hat.

![System → Einstellungen → Jira: Server, Auth, Import, automatisches Anlegen und Benutzerfelder.](../img/system-settings-jira.png)

## Aktivieren + Ziel

Der Import ist **standardmÃ¤Ãig aus** (er legt Daten an). Unter System â Einstellungen â Jira:

- `jira.import_enabled` = an
- `jira.import_project` / `jira.import_activity` = das **Standard**-Ziel (siehe
  [projektbezogenes Routing](project-routing.md) und [automatisches Anlegen](auto-create.md), um
  verschiedene Jira-Projekte an verschiedene Kimai-Projekte zu senden)
- `jira.import_window_days` = wie weit zurÃ¼ck gesucht wird (Standard 14)

Aus einem eigenen Cron-Eintrag ausfÃ¼hren, getrennt vom Abgleich:

```bash
0 * * * * cd /path/to/kimai && bin/console kimai:jira:import >> var/log/jira-cron.log 2>&1
```

## Verhalten

- Findet die im Zeitfenster gebuchten VorgÃ¤nge jedes Benutzers, importiert dessen eigene Worklogs
  und speichert am Zeiteintrag den Jira-SchlÃ¼ssel, die Worklog-ID und `jira_sync_status=synced`.
- **Dedupliziert** anhand bereits gespeicherter Worklog-IDs â erneute LÃ¤ufe erzeugen nie
  Duplikate, und es wird nie ein Worklog erneut importiert, das die eigene AuswÃ¤rts-Sync von Kimai
  erzeugt hat.
- **Zeitzonen-korrekt** â der Jira-Zeitpunkt `started` wird exakt in die Zeitzone des Benutzers
  Ã¼berfÃ¼hrt.
- **Routet** anhand des Jira-SchlÃ¼ssels, legt optional Projekte **automatisch an** und Ã¼bernimmt
  zugeordnete [Benutzerfelder](custom-fields.md).
- `--dry-run` meldet ohne zu schreiben; `--user=ID` beschrÃ¤nkt auf einen Benutzer; `--days=N`
  Ã¼berschreibt das Zeitfenster.

## Bekannte EinschrÃ¤nkung (MVP)

Importierte EintrÃ¤ge werden direkt gespeichert (um eine Sync-RÃ¼ckkopplungsschleife zu vermeiden),
was Kimais Satz-Berechnung umgeht â sie starten daher mit **Null-/StandardsÃ¤tzen**. Bearbeiten Sie
einen Eintrag oder lassen Sie Ihre Ã¼blichen Satzregeln beim nÃ¤chsten Speichern greifen.

Siehe auch: [Projektbezogenes Routing](project-routing.md) Â· [Automatisches Anlegen](auto-create.md)
Â· [Benutzerfelder](custom-fields.md).
