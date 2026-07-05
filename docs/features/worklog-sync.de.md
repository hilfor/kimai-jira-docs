# Worklog-Sync (Kimai → Jira)

Die Kernfunktion: einen Zeiteintrag mit einem Jira-Vorgang verknüpfen und die erfasste Zeit als
Worklog nach Jira übertragen.

## Das Jira-Vorgang-Feld

Jeder Zeiteintrag (Erstellen-/Bearbeiten-Formular und die Start/Stopp-Leiste) hat ein optionales
**Jira-Vorgang**-Feld (z. B. `PROJ-123`, geprüft gegen `^[A-Z][A-Z0-9_]*-\d+$`). Leer lassen, um
den Eintrag von Jira fernzuhalten. Als Zeiteintrags-Spalte wird es als **anklickbarer Link** zum
Ticket dargestellt, und der Schlüssel wird während der Eingabe **live gegen Jira geprüft** (zeigt
die Vorgangs-Zusammenfassung oder einen „nicht gefunden“-Hinweis) – nur beratend, es blockiert das
Speichern nie.

Sie müssen nicht den reinen Schlüssel eintippen: **fügen Sie ein, was Sie haben** – es wird beim
Speichern auf den kanonischen Schlüssel normalisiert. Eine vollständige Ticket-URL
(`https://ihr-jira/browse/PROJ-123?jql=…`), ein Board- oder Backlog-Link
(`…?selectedIssue=PROJ-123`, `…?issueKey=OPS-7`) oder ein flüchtig getippter Schlüssel
(Kleinschreibung, überflüssige Leerzeichen) werden alle zu `PROJ-123`. Text ohne erkennbaren
Schlüssel bleibt unverändert, damit die Formatprüfung ihn weiterhin abfängt.

## Wann ein Worklog erstellt wird

Ein Worklog wird erstellt/aktualisiert, sobald der Eintrag **sowohl** eine Endzeit als auch einen
Vorgangsschlüssel hat **und** der Benutzer ein Token besitzt. Ein laufender Timer wird nie
zwischendurch synchronisiert. Ändert man den Schlüssel, nachdem ein Worklog existiert, wird das
alte gelöscht und ein neues unter dem neuen Schlüssel angelegt. Der Worklog-Kommentar ist die
Beschreibung des Zeiteintrags (per `jira.sync_comment` umschaltbar).

## Kimai ist die maßgebliche Quelle

Speichern/Stoppen wartet nie auf Jira – der HTTP-Aufruf wird nach der Antwort verzögert
ausgeführt, mit knappen Zeitlimits. Vorübergehende Fehler (Netzwerk, `5xx`, `429`) werden mit
Backoff wiederholt; dauerhafte (`401`/`403`/`404`/`400`) erscheinen in der Sync-Status-Spalte und
pausieren bei einem `401` die Jira-Aufrufe dieses Benutzers, bis das Token korrigiert ist. Ein
Hintergrund-Abgleich (`kimai:jira:sync`, per Cron) trägt alles nach, was inline nicht
synchronisiert werden konnte.

## Einrichtung

Siehe die Haupt-[Startseite](../index.md) für die Instanz-Einstellungen (Server-URL, Auth-Modus,
Sync-Modus), die benutzereigene Token-Seite und den Cron-Eintrag für `kimai:jira:sync`.

Siehe auch: [Import](importing.md) · [Benachrichtigungen](notifications.md) ·
[Fehlerbehebung](troubleshooting.md).
