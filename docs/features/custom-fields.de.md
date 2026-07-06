# Benutzerfeld-Übernahme

Übernehmen Sie ausgewählte Jira-**Benutzerfelder** (eine Kostenstelle, eine Kundenreferenz, …) auf
importierte Zeiteinträge, damit ein bereits in Jira vorhandener Wert in Kimai landet, statt neu
eingetippt zu werden.

## Zuordnung konfigurieren

Auf dem **Kunden-Bearbeitungsformular** (Kunden → Kunde bearbeiten → Jira) die Option
`jira_import_fields` für diesen Kunden setzen – eine Zuordnung pro Zeile:

```text
customfield_10012 = cost_centre
duedate = due_date
```

- **Links** = die Jira-Feld-ID (`customfield_<n>`, oder ein Systemfeld wie `duedate`). Sie müssen
  eine ID selten von Hand nachschlagen: Mit einem gültigen Jira-Token listet ein Helfer
  **Jira-Feld einfügen…** unterhalb des Zuordnungsfelds Ihre Jira-Felder (ID + Name) und hängt die
  Zeile `customfield_… = ` für Sie an. Die ID finden Sie auch in Jira unter *Einstellungen →
  Vorgänge → Benutzerdefinierte Felder* oder über `GET /rest/api/2/field`.
- **Rechts** = der Name des **Kimai-Zeiteintrag-Benutzerfelds**, in das kopiert wird. Er muss
  klein geschrieben sein (Buchstaben, Ziffern, Unterstrich) und darf nicht mit `jira_` beginnen
  (reserviert). Legen Sie das passende Kimai-Benutzerfeld zuerst an (System → Benutzerfelder),
  damit es ein Ziel gibt.
- `#`-Kommentare sowie leere/ungültige Zeilen werden ignoriert; bei doppelter Jira-ID gewinnt die
  letzte Zeile.

## Nur skalare Werte – und übersprungene Felder werden sichtbar gemacht

Nur **einfache Werte** lassen sich sauber übernehmen: Text, Zahlen und Ja/Nein (als `1`/`0`
gespeichert). Datumsangaben kommen als Text an und werden unverändert übernommen.

Komplexere Jira-Feldtypen – Einfachauswahl (`{value: …}`), Benutzer (`{displayName: …}`),
Mehrfachauswahl (eine Liste) – werden **nicht** importiert. Statt fehlzuschlagen oder zu raten,
**überspringt** der Importer sie und **teilt es Ihnen mit**, auf allen Kanälen:

- dem **Dashboard-Widget** („2 Jira-Felder konnten nicht importiert werden“),
- einem **Admin-Banner** beim nächsten Seitenaufruf,
- der **wöchentlichen Zusammenfassung** per E-Mail,
- und der Ausgabe des Befehls `kimai:jira:import` (`--dry-run` sagt sie ebenfalls voraus).

Jede Warnung benennt das genaue Feld und seinen Typ – wenn Sie also immer wieder auf eines stoßen
(eine Kostenstelle als Auswahlliste ist häufig), wissen Sie genau, was Sie als Nächstes anfragen
sollten. Das Beheben oder Entfernen einer Zuordnung löscht ihre Warnung beim nächsten Lauf
automatisch.

Siehe auch: [Import](importing.md) · [Benachrichtigungen](notifications.md).
