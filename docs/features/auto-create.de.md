# Automatisches Anlegen (Opt-in)

Wenn Sie Zeit auf viele Jira-Tickets buchen, ist es mühsam, für jedes vorab ein Kimai-Projekt
anzulegen. Das automatische Anlegen lässt **Struktur aus der tatsächlichen Arbeit entstehen**: Ein
Jira-Schlüssel, den noch kein Projekt beansprucht, erhält beim Import ein eigens dafür angelegtes
Kimai-Projekt.

## Aktivieren

Auf dem **Kunden-Bearbeitungsformular** (Kunden → Kunde bearbeiten → Jira):

- `jira_autocreate_enabled` = **an** (Standard **aus** – es werden Daten angelegt).

Es gibt keine separate Kundeneinstellung zur Auswahl: Automatisch angelegte Projekte werden unter
**dem Kunden abgelegt, dessen Konfiguration den Import ausgelöst hat** (dessen eigene Jira).
Außerdem benötigen Sie weiterhin die `jira_import_activity` dieses Kunden – das automatische Anlegen
erzeugt *Projekte*, keine Tätigkeiten, daher verwenden importierte Einträge die Importtätigkeit des
Kunden, bis Sie einen projektbezogenen Override setzen.

## Was beim Import passiert

Für einen Jira-Schlüssel, den kein vorhandenes Projekt [beansprucht](project-routing.md):

1. Ein Kimai-Projekt wird angelegt, **benannt nach dem Jira-Projekt** (aus dem Jira-Projektnamen;
   ersatzweise der bloße Schlüssel), abgelegt unter **diesem Kunden** (dem gerade importierten).
2. Das Projekt **beansprucht diesen Schlüssel** bereits (sein Feld `Jira-Projektschlüssel` ist
   vorausgefüllt), sodass es ab dem nächsten Lauf ein normales, geroutetes Projekt ist, das Sie
   umbenennen, umhängen oder mit Sätzen versehen können.
3. Die importierten Worklogs werden darunter abgelegt (mit der Importtätigkeit des Kunden).

## Garantien

- **Nur mit Opt-in** – andernfalls fällt der Importer auf das Standardziel des Kunden zurück und
  protokolliert den Grund.
- **Idempotent** – ein Projekt pro Jira-Schlüssel pro Lauf, und erneute Läufe finden die
  bestehende Beanspruchung, statt Duplikate anzulegen.
- **Nie zerstörerisch / stürzt nie ab** – ein fehlgeschlagenes Anlegen fällt auf das Standardziel
  des Kunden zurück und wird protokolliert; der Lauf läuft weiter.
- **`--dry-run`** meldet die Projekte, die es anlegen *würde*, ohne etwas zu schreiben.
- Es werden **keine Tätigkeiten** automatisch angelegt.

Siehe auch: [Projektbezogenes Routing](project-routing.md) · [Import](importing.md).
