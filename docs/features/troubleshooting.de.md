# Fehlerbehebung

Support-Anfragen beginnen mit dem plugin-eigenen Log: `var/log/jira-<env>-<date>.log` (ein eigener
`jira`-Monolog-Kanal; das Token wird nie protokolliert).

## Es wird nichts nach Jira synchronisiert

- **Kein Token / Token ungültig** – prüfen Sie die benutzereigene Jira-Einstellungsseite (Status
  `valid` / `invalid`) und das Dashboard-Widget. Ein `401` pausiert die Jira-Aufrufe dieses
  Benutzers, bis das Token aktualisiert ist.
- **Eintrag ohne Endzeit oder ohne Vorgangsschlüssel** – ein Worklog wird erst erstellt, wenn
  beides vorhanden ist.
- **`sync_mode = manual`** – nichts wird inline synchronisiert; führen Sie
  `kimai:jira:sync --status=pending` aus.
- **Jira nicht erreichbar** – Einträge bleiben `pending`; der Abgleich trägt sie nach. Prüfen Sie,
  ob der Circuit Breaker nicht offen ist (wiederholte Verbindungsfehler pausieren Inline-Versuche
  für einige Minuten).

## Der Importer legt nichts an

- `jira.import_enabled` ist aus, oder es ist kein Ziel auflösbar – ohne projektbezogenes
  [Routing](project-routing.md), ohne [automatisches Anlegen](auto-create.md) und mit
  ungesetztem/gelöschtem Standardprojekt oder -tätigkeit meldet der Lauf „nicht konfiguriert“ und
  beendet sich.
- Führen Sie `bin/console kimai:jira:import --dry-run` aus, um pro Benutzer und pro Vorgang zu
  sehen, was er *tun würde*.

## Importierte Zeit landet im falschen Projekt

- Prüfen Sie, welches Projekt diesen Jira-Schlüssel [beansprucht](project-routing.md) (sein Feld
  `Jira-Projektschlüssel`). Eine **doppelte Beanspruchung** (zwei Projekte, gleicher Schlüssel)
  wird zur niedrigeren Projekt-ID aufgelöst und protokolliert – beheben Sie das Duplikat.
- Ein nicht beanspruchter Schlüssel verwendet den globalen Standard (oder legt automatisch an,
  falls aktiviert).

## Ein Benutzerfeld wird nicht importiert

- Nur [skalare Werte](custom-fields.md) (Text/Zahl/Ja-Nein) werden übernommen. Auswahllisten,
  Benutzer und Mehrfachwert-Felder werden übersprungen – prüfen Sie Dashboard-Widget /
  Admin-Banner / Zusammenfassung, die das genaue Feld und den Typ benennen.
- Die Kimai-Seite muss ein echtes Zeiteintrag-Benutzerfeld sein (System → Benutzerfelder) mit
  einem klein geschriebenen Namen, der nicht mit `jira_` beginnt.

## E-Mails kommen nicht an

- Ein funktionierendes `MAILER_DSN` ist erforderlich, ebenso `ext-xsl` (Kimai-Core benötigt es, um
  überhaupt Mail zu rendern).
- Setzen Sie `framework.router.default_uri`, damit Links in per Cron versandter Mail auf Ihre echte
  Domain zeigen, nicht auf `localhost`. Fehler werden im `jira`-Kanal protokolliert und bringen den
  Lauf nie zum Absturz.

Siehe auch: [Benachrichtigungen](notifications.md) · [Startseite](../index.md).
