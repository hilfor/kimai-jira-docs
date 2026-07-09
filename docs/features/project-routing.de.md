# Projektbezogenes Routing

Standardmäßig legt der Importer jedes importierte Worklog unter einem einzigen Projekt ab (der
`jira_import_project` des Kunden). Das ist falsch, wenn die Jira-Instanz eines Kunden **viele
Projekte** hat – dann soll `PROJ-123` unter das eine Kimai-Projekt und `OPS-9` unter ein anderes.
Genau das leistet das projektbezogene Routing.

![Die Felder „Jira-Projektschlüssel“ und „Jira-Importtätigkeit“ im Bearbeitungsformular eines Kimai-Projekts.](../img/project-jira-keys.png)

## Funktionsweise

Jedes Kimai-Projekt **beansprucht** die Jira-Projektschlüssel, die zu ihm gehören. Öffnen Sie das
normale Bearbeitungsformular eines Projekts (Administration → Projekte → *bearbeiten*) und tragen
Sie ein:

- **Jira-Projektschlüssel** – ein oder mehrere Jira-Schlüssel in Großbuchstaben, kommagetrennt
  (z. B. `PROJ` oder `PROJ, OPS`). Jedes importierte Worklog eines Vorgangs wie `PROJ-123` wird
  unter diesem Projekt abgelegt.
- **Jira-Importtätigkeit** *(optional)* – die Tätigkeit, die importierte Einträge dieses Projekts
  verwenden; sie überschreibt die `jira_import_activity` des Kunden. Leer lassen für den Standard
  des Kunden.

Die Zuordnung liegt **am Projekt selbst**, wird also mit dem Projekt angelegt und gelöscht – es
gibt keine separate Tabelle oder Seite, die synchron gehalten werden müsste.

!!! tip "Autovervollständigung"
    Mit einem gültigen Jira-Token (aus Ihren **Jira-Einstellungen**) schlägt das Feld
    **Jira-Projektschlüssel** während der Eingabe Ihre Jira-Projekte vor (`KEY — Name`), sodass
    Sie sich die genauen Schlüssel nicht merken müssen. Ohne Token bleibt es ein einfaches
    Textfeld – die Funktion ist reine Komfortfunktion.

## Auflösungsreihenfolge

Für jeden importierten Vorgangsschlüssel wählt der Importer das Ziel in dieser Reihenfolge:

1. **Ein Projekt, das den Schlüssel beansprucht** → dieses Projekt (+ dessen Tätigkeits-Override
   oder die Importtätigkeit des Kunden).
2. **Kein Projekt beansprucht ihn** → [automatisches Anlegen](auto-create.md) eines Projekts
   (falls aktiviert), sonst der Standard des Kunden `jira_import_project` / `jira_import_activity`.
3. **Ein Routing-Fehler** – ein Ziel wurde gefunden, ist aber nicht verwendbar. Zwei
   unterschiedliche Fälle, beide werden gemeldet (protokolliert und in der Lauf-Zusammenfassung
   gezählt), niemals stillschweigend verworfen:
    - Ein Projekt **beansprucht den Schlüssel, hat aber keine Importtätigkeit** (kein
      projektbezogener Override und keine Standard-Tätigkeit des Kunden). Das ist ein Fehler des
      *beanspruchenden* Projekts, keine fehlende Beanspruchung – es fehlt die Tätigkeit, also
      beheben Sie es, indem Sie eine Importtätigkeit hinterlegen.
    - Ein **nicht beanspruchter Schlüssel ohne Standardprojekt** (kein Projekt beansprucht den
      Schlüssel, automatisches Anlegen ist aus, und der Kunde hat kein `jira_import_project`). Das
      unterscheidet sich von Fall 2: Ein nicht beanspruchter Schlüssel mit konfiguriertem Standard
      fällt einfach auf diesen Standard zurück – erst ein *fehlender* Standard macht daraus einen
      Fehler.

**Abwärtskompatibel:** Beansprucht kein Projekt einen Schlüssel, verwendet jedes Worklog den
Standard des Kunden – exakt das Verhalten mit einem Ziel wie vor dieser Funktion.

## Hinweise

- Schlüssel werden ohne Beachtung der Groß-/Kleinschreibung verglichen und müssen wie ein
  Jira-Projektschlüssel aussehen (`^[A-Z][A-Z0-9_]*$`); ungültige Einträge werden ignoriert.
- **Doppelte Beanspruchung** – beanspruchen zwei Projekte denselben Schlüssel, gewinnt die
  niedrigere Projekt-ID; der Konflikt wird protokolliert und gezählt, damit Sie ihn erkennen und
  beheben können.
- Ein Schlüssel, dessen Projekt zwischenzeitlich gelöscht/ausgeblendet wurde, fällt in diesem Lauf
  auf den Standard des Kunden zurück.

Siehe auch: [Import](importing.md) · [Automatisches Anlegen](auto-create.md).
