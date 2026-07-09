# Fehlerbehebung

Support-Anfragen beginnen mit dem plugin-eigenen Log: `var/log/jira-<env>-<date>.log` (ein eigener
`jira`-Monolog-Kanal; das Token wird nie protokolliert).

## Jira-Funktionen funktionieren nicht mehr

Sync, Import und Vorgangssuche **setzen gemeinsam aus**, wenn die Lizenz inaktiv oder veraltet ist –
während die Kimai-Zeiterfassung weiterläuft. Ist alles rund um Jira gleichzeitig verstummt,
verdächtigen Sie zuerst die Lizenz.

- **Lizenzstatus prüfen.** Achten Sie auf das Admin-Banner: einen Kulanz-Countdown („*sync keeps
  working for N more day(s)*"), einen „inaktiv"-Hinweis oder „kein Lizenzschlüssel konfiguriert".
  Bestätigen Sie dann den Schlüssel unter **System → Einstellungen → Jira** (oder die Umgebungsvariable
  `JIRA_LICENSE_KEY`, die **Vorrang** vor dem Einstellungsfeld hat – prüfen Sie beide). Siehe
  [Lizenz](../licensing.md).
- **Den `kimai:jira:sync`-Cron prüfen.** Der tägliche Lizenz-Heartbeat läuft über diesen Cron und
  ist das Einzige, das die Offline-Veraltungs-Uhr zurücksetzt. Ist er nicht eingeplant, degradiert
  eine bezahlte, online betriebene Installation von selbst nach ~44 Tagen (30 Tage
  Offline-Veraltung + 14 Tage Kulanz). Planen Sie ihn ein (siehe [Einrichtung → Cron](../configure.md))
  oder setzen Sie `JIRA_LICENSE_OFFLINE_GRACE_DAYS=0` für eine bewusst air-gapped Installation.
- **Das Log prüfen.** Im `jira`-Kanal (`var/log/jira-<env>-<date>.log`) protokolliert ein pausierter
  Lauf `Reconciler skipped: Jira subscription inactive.` (Sync) bzw. `Import skipped: Jira
  subscription inactive.` (Import). Beim manuellen Aufruf der Befehle erscheint `Jira subscription
  inactive — … Enter a valid license key under Settings → Jira.` Ein Aussetzer des Lizenz-Endpunkts
  protokolliert `Jira license status ping failed; failing open.` auf Info-Ebene und ist harmlos – er
  deaktiviert eine funktionierende Instanz nie.

Sobald ein gültiger Schlüssel aktiv ist, führen Sie `bin/console kimai:jira:sync` aus – der
Heartbeat aktiviert die Funktionen wieder, und in der Warteschlange stehende Worklogs werden
nachgetragen.

## Es wird nichts nach Jira synchronisiert

- **Kein Token / Token für diesen Kunden ungültig** – öffnen Sie die kundenbezogene
  Token-Übersicht (**Jira-Einstellungen** → Zeile des Kunden, bzw. `/jira/settings/{user}`) und
  prüfen Sie den Status dieses Kunden (`valid` / `invalid`) sowie die kundenbezogene Aufschlüsselung
  im Dashboard-Widget. Ein `401` pausiert die Jira-Aufrufe dieses Benutzers für diesen Kunden, bis
  das Token aktualisiert ist.
- **Das Projekt hat keinen Kunden, oder der Kunde hat keine Jira konfiguriert** – die
  Synchronisierung richtet sich nach dem Kunden des Zeiteintrags (Zeiteintrag → Projekt → Kunde).
  Ein Eintrag, dessen Projekt keinen Kunden hat oder dessen Kunde keine `jira_server_url` gesetzt
  hat, wird nie synchronisiert.
- **Eintrag ohne Endzeit oder ohne Vorgangsschlüssel** – ein Worklog wird erst erstellt, wenn
  beides vorhanden ist.
- **`sync_mode = manual`** – nichts wird inline synchronisiert; führen Sie
  `kimai:jira:sync --status=pending` aus.
- **Jira nicht erreichbar** – Einträge bleiben `pending`; der Abgleich trägt sie nach. Prüfen Sie,
  ob der Circuit Breaker nicht offen ist (wiederholte Verbindungsfehler pausieren Inline-Versuche
  für einige Minuten).

## Der Importer legt nichts an

- Kein Kunde hat `jira_import_enabled` an, oder es ist kein Ziel auflösbar – ohne projektbezogenes
  [Routing](project-routing.md), ohne [automatisches Anlegen](auto-create.md) und mit
  ungesetztem/gelöschtem Standardprojekt oder -tätigkeit **an diesem Kunden** meldet der Lauf „nicht
  konfiguriert“ und beendet sich.
- Führen Sie `bin/console kimai:jira:import --dry-run` aus, um pro Benutzer und pro Vorgang zu
  sehen, was er *tun würde*.

## Importierte Zeit landet im falschen Projekt

- Prüfen Sie, welches Projekt diesen Jira-Schlüssel [beansprucht](project-routing.md) (sein Feld
  `Jira-Projektschlüssel`). Eine **doppelte Beanspruchung** (zwei Projekte, gleicher Schlüssel)
  wird zur niedrigeren Projekt-ID aufgelöst und protokolliert – beheben Sie das Duplikat.
- Ein nicht beanspruchter Schlüssel verwendet das Standard-Importziel des Kunden (oder legt
  automatisch unter diesem Kunden an, falls aktiviert).

## Ein Benutzerfeld wird nicht importiert

- Nur [skalare Werte](custom-fields.md) (Text/Zahl/Ja-Nein) werden übernommen. Auswahllisten,
  Benutzer und Mehrfachwert-Felder werden übersprungen – prüfen Sie Dashboard-Widget /
  Admin-Banner / Zusammenfassung, die das genaue Feld und den Typ benennen.
- Die Kimai-Seite muss ein echtes Zeiteintrag-Benutzerfeld sein (System → Benutzerfelder) mit
  einem klein geschriebenen Namen, der nicht mit `jira_` beginnt.

## Ein Jira-Projekt oder -Feld fehlt in einer Auswahlliste

- Die Auswahllisten für **Projektschlüssel** und **Benutzerfelder** beziehen ihre Optionen aus
  Jira, doch die Antwort wird **pro Benutzer für ~10 Minuten zwischengespeichert**, damit die
  Formulare schnell bleiben. Ein Projekt, Feld oder eine Berechtigung, die Sie gerade in Jira
  geändert haben, kann bis zu dieser Zeit brauchen, um zu erscheinen. Warten Sie ab, oder führen
  Sie `bin/console cache:clear` aus, um die zwischengespeicherten Abfragen sofort zu verwerfen.
- Erscheint es nie, liegt es nicht am Cache – das Token kann es nicht sehen: prüfen Sie, ob das
  Konto Berechtigung auf dieses Jira-Projekt hat, und öffnen Sie das Formular erneut.

## E-Mails kommen nicht an

- Ein funktionierendes `MAILER_DSN` ist erforderlich, ebenso `ext-xsl` (Kimai-Core benötigt es, um
  überhaupt Mail zu rendern).
- Setzen Sie `framework.router.default_uri`, damit Links in per Cron versandter Mail auf Ihre echte
  Domain zeigen, nicht auf `localhost`. Fehler werden im `jira`-Kanal protokolliert und bringen den
  Lauf nie zum Absturz.

Siehe auch: [Benachrichtigungen](notifications.md) · [Startseite](../index.md).
