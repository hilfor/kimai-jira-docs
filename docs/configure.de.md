# Einrichtung

## Erste Schritte

Jira wird **pro Kunde** konfiguriert – jeder Kimai-Kunde verweist auf seine eigene Jira-Instanz,
sodass ein Unternehmen, das zwei Klienten auf zwei getrennten Jira-Servern betreut, jeden Kunden
unabhängig einrichtet.

1. Als Administrator einen Kunden öffnen (**Kunden → Kunde bearbeiten → Jira**) und die Jira-
   **Server-URL** sowie den **Auth-Modus** dieses Kunden eintragen (dazu die gewünschten Sync- und
   Import-Optionen).
2. Jeder Benutzer öffnet **Jira-Einstellungen** in seinem Benutzermenü – eine Übersicht listet jeden
   Kunden, für den er ein Token halten kann – wählt einen Kunden und fügt sein eigenes Token für die
   Jira **dieses Kunden** ein. Eine Schaltfläche **Verbindung testen** meldet ein konkretes Ergebnis
   (abgelehnte Zugangsdaten / DNS / TLS / Zeitüberschreitung). Ein Benutzer, der Zeit für zwei Kunden
   erfasst, hält zwei Token – eines pro Kunde.
3. Das **Jira-Vorgang**-Feld an einem Zeiteintrag ausfüllen – das Worklog erscheint, sobald der
   Eintrag mit Endzeit gestoppt oder gespeichert wird. Es wird zur Jira des **Kunden** des
   Zeiteintrags synchronisiert (Zeiteintrag → Projekt → Kunde), mit dem Token dieses Benutzers für
   diesen Kunden.

![Die Jira-Einstellungen eines Kunden im Kunden-Bearbeitungsformular: Server-URL,
Authentifizierungsmethode, Worklog-Synchronisation und die Import-Zieloptionen.](img/system-settings-jira.png)
*Schritt 1 – ein Administrator konfiguriert Jira am Kunden. Jeder Kunde kann auf eine andere Jira zeigen.*

![Die benutzereigene Jira-Einstellungsübersicht: eine Zeile pro Kunde mit Token-Status, jeweils
verlinkt auf eine kundenbezogene Bearbeitungsseite.](img/settings-page.png)
*Jeder Benutzer verwaltet hier sein **eigenes** Token je Kunde – jedes wird verschlüsselt
gespeichert und nie zurückgezeigt.*

![Die kundenbezogene Token-Seite: maskierter Token-Hinweis, Verbindungsstatus, Zeitpunkt der letzten
Prüfung und die Schaltflächen „Verbindung testen“ / „Token löschen“ / „Token speichern“.](img/settings-token-edit.png)
*Schritt 2 – ein Benutzer fügt sein Token für einen Kunden ein; der Status wird beim Speichern, per
„Verbindung testen“ und durch einen täglichen Heartbeat geprüft.*

## Authentifizierung

Der Auth-Modus wird **pro Kunde** festgelegt (`jira_auth_mode` am jeweiligen Kunden), da er davon
abhängt, welche Art von Jira dieser Kunde betreibt:

- **Jira Server / Data Center** – `jira_auth_mode = bearer`; jeder Benutzer fügt einen
  **persönlichen Zugriffstoken** ein (Jira-Profil → Personal Access Tokens) für diesen Kunden.
- **Jira Cloud** – `jira_auth_mode = basic`; jeder Benutzer fügt einen **API-Token** ein
  ([id.atlassian.com](https://id.atlassian.com/manage-profile/security/api-tokens)) für diesen
  Kunden und trägt seine Jira-Konto-E-Mail in den Kimai-Einstellungen ein (Abschnitt „Jira“).

Die Server-URL ist eine Sicherheitsgrenze – das Token einer Person wird an den Host gesendet, den
der Kunde angibt – und wird daher **beim Speichern des Kunden validiert**: Sie muss `https://`
verwenden (Token werden nie im Klartext übertragen) und darf nicht auf eine private, Loopback- oder
Link-local-Adresse zeigen. Eine Änderung der Kunden-URL lässt beim nächsten täglichen Heartbeat
jedes gespeicherte Token dieses Kunden neu validieren, da ein gegen den alten Host geprüftes Token
nichts über einen neuen aussagt.

## Cron

Der Abgleich und der (optionale) Importer laufen per Cron – als der Benutzer eintragen, unter dem
Kimais eigene Cron-Jobs laufen:

```bash
# Abgleich: trägt alles nach, was inline nicht synchronisiert werden konnte (alle 5 Minuten)
*/5 * * * * cd /path/to/kimai && bin/console kimai:jira:sync   >> var/log/jira-cron.log 2>&1

# Importer: holt die eigenen Jira-Worklogs jedes Benutzers nach Kimai (Opt-in; z. B. stündlich)
0   * * * * cd /path/to/kimai && bin/console kimai:jira:import  >> var/log/jira-cron.log 2>&1

# Wöchentliche Admin-Zusammenfassung (Montagmorgen)
0 8 * * 1   cd /path/to/kimai && bin/console kimai:jira:sync --digest >> var/log/jira-cron.log 2>&1
```

Damit per Cron versandte E-Mails auf Ihre Instanz verweisen, setzen Sie `framework.router.default_uri`
in Ihrer Kimai-Konfiguration, sodass Links auf Ihre echte Domain statt auf `localhost` zeigen.

## Referenz der kundenbezogenen Felder

Jede Jira-Einstellung liegt auf dem **Kunden-Bearbeitungsformular** (Kunden → Kunde bearbeiten →
Jira). Jeder Kunde wird unabhängig konfiguriert. Was jedes Feld bewirkt, ist in den Anleitungen
beschrieben:

- `jira_server_url`, `jira_auth_mode`, `jira_sync_mode`, `jira_delete_remote`, `jira_sync_comment`
  – Verbindung und Auswärts-Sync-Verhalten ([Worklog-Sync](features/worklog-sync.md)).
- `jira_import_enabled`, `jira_import_project`, `jira_import_activity`, `jira_import_window_days` –
  das Ziel des aktivierbaren Imports ([Import](features/importing.md)); projektbezogenes Routing und
  automatisches Anlegen senden verschiedene Jira-Projekte an verschiedene Kimai-Projekte
  ([Projektbezogenes Routing](features/project-routing.md),
  [Automatisches Anlegen](features/auto-create.md)).
- `jira_autocreate_enabled` – Projekte automatisch unter diesem Kunden anlegen
  ([Automatisches Anlegen](features/auto-create.md)).
- `jira_import_fields` – Benutzerfeld-Zuordnung ([Benutzerfeld-Übernahme](features/custom-fields.md)).
- Benachrichtigungen – [Benachrichtigungen & Sichtbarkeit](features/notifications.md).
