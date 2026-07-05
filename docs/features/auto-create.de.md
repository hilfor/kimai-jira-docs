# Automatisches Anlegen (Opt-in)

Wenn Sie Zeit auf viele Jira-Tickets buchen, ist es mühsam, für jedes vorab ein Kimai-Projekt
anzulegen. Das automatische Anlegen lässt **Struktur aus der tatsächlichen Arbeit entstehen**: Ein
Jira-Schlüssel, den noch kein Projekt beansprucht, erhält beim Import ein eigens dafür angelegtes
Kimai-Projekt.

## Aktivieren

Unter **System → Einstellungen → Jira**:

- `jira.autocreate_enabled` = **an** (Standard **aus** – es werden Daten angelegt).
- `jira.autocreate_customer` = der Kunde, unter dem neue Projekte abgelegt werden. Das
  automatische Anlegen bleibt aus, bis dies gesetzt ist.

Außerdem benötigen Sie weiterhin eine globale `jira.import_activity` – das automatische Anlegen
erzeugt *Projekte*, keine Tätigkeiten, daher verwenden importierte Einträge die globale Tätigkeit,
bis Sie einen projektbezogenen Override setzen.

## Was beim Import passiert

Für einen Jira-Schlüssel, den kein vorhandenes Projekt [beansprucht](project-routing.md):

1. Ein Kimai-Projekt wird angelegt, **benannt nach dem Jira-Projekt** (aus dem Jira-Projektnamen;
   ersatzweise der bloße Schlüssel), abgelegt unter dem konfigurierten Kunden.
2. Das Projekt **beansprucht diesen Schlüssel** bereits (sein Feld `Jira-Projektschlüssel` ist
   vorausgefüllt), sodass es ab dem nächsten Lauf ein normales, geroutetes Projekt ist, das Sie
   umbenennen, umhängen oder mit Sätzen versehen können.
3. Die importierten Worklogs werden darunter abgelegt (mit der globalen Importtätigkeit).

## Garantien

- **Nur mit Opt-in** und erfordert einen Kunden – andernfalls fällt der Importer auf den globalen
  Standard zurück und protokolliert den Grund.
- **Idempotent** – ein Projekt pro Jira-Schlüssel pro Lauf, und erneute Läufe finden die
  bestehende Beanspruchung, statt Duplikate anzulegen.
- **Nie zerstörerisch / stürzt nie ab** – ein fehlgeschlagenes Anlegen fällt auf den Standard
  zurück und wird protokolliert; der Lauf läuft weiter.
- **`--dry-run`** meldet die Projekte, die es anlegen *würde*, ohne etwas zu schreiben.
- Es werden **keine Tätigkeiten** automatisch angelegt.

Siehe auch: [Projektbezogenes Routing](project-routing.md) · [Import](importing.md).
