# Projektbezogenes Routing

StandardmÃ¤Ãig legt der Importer jedes importierte Worklog unter einem einzigen Projekt ab
(`jira.import_project`). Das ist falsch, wenn Ihre Jira-Instanz **viele Projekte** hat â dann
soll `PROJ-123` unter das eine Kimai-Projekt und `OPS-9` unter ein anderes. Genau das leistet das
projektbezogene Routing.

![Die Felder „Jira-Projektschlüssel“ und „Jira-Importtätigkeit“ im Bearbeitungsformular eines Kimai-Projekts.](../img/project-jira-keys.png)

## Funktionsweise

Jedes Kimai-Projekt **beansprucht** die Jira-ProjektschlÃ¼ssel, die zu ihm gehÃ¶ren. Ãffnen Sie das
normale Bearbeitungsformular eines Projekts (Administration â Projekte â *bearbeiten*) und tragen
Sie ein:

- **Jira-ProjektschlÃ¼ssel** â ein oder mehrere Jira-SchlÃ¼ssel in GroÃbuchstaben, kommagetrennt
  (z. B. `PROJ` oder `PROJ, OPS`). Jedes importierte Worklog eines Vorgangs wie `PROJ-123` wird
  unter diesem Projekt abgelegt.
- **Jira-ImporttÃ¤tigkeit** *(optional)* â die TÃ¤tigkeit, die importierte EintrÃ¤ge dieses Projekts
  verwenden; sie Ã¼berschreibt die globale `jira.import_activity`. Leer lassen fÃ¼r den globalen
  Standard.

Die Zuordnung liegt **am Projekt selbst**, wird also mit dem Projekt angelegt und gelÃ¶scht â es
gibt keine separate Tabelle oder Seite, die synchron gehalten werden mÃ¼sste.

## AuflÃ¶sungsreihenfolge

FÃ¼r jeden importierten VorgangsschlÃ¼ssel wÃ¤hlt der Importer das Ziel in dieser Reihenfolge:

1. **Ein Projekt, das den SchlÃ¼ssel beansprucht** â dieses Projekt (+ dessen TÃ¤tigkeits-Override
   oder die globale TÃ¤tigkeit).
2. **Kein Projekt beansprucht ihn** â [automatisches Anlegen](auto-create.md) eines Projekts
   (falls aktiviert), sonst der globale Standard `jira.import_project` / `jira.import_activity`.
3. **Nichts auflÃ¶sbar** (nicht beanspruchter SchlÃ¼ssel, kein Standard) â der Vorgang wird
   Ã¼bersprungen und in der Lauf-Zusammenfassung gezÃ¤hlt, niemals stillschweigend verworfen.

**AbwÃ¤rtskompatibel:** Beansprucht kein Projekt einen SchlÃ¼ssel, verwendet jedes Worklog den
globalen Standard â exakt das Verhalten mit einem Ziel wie vor dieser Funktion.

## Hinweise

- SchlÃ¼ssel werden ohne Beachtung der GroÃ-/Kleinschreibung verglichen und mÃ¼ssen wie ein
  Jira-ProjektschlÃ¼ssel aussehen (`^[A-Z][A-Z0-9_]*$`); ungÃ¼ltige EintrÃ¤ge werden ignoriert.
- **Doppelte Beanspruchung** â beanspruchen zwei Projekte denselben SchlÃ¼ssel, gewinnt die
  niedrigere Projekt-ID; der Konflikt wird protokolliert und gezÃ¤hlt, damit Sie ihn erkennen und
  beheben kÃ¶nnen.
- Ein SchlÃ¼ssel, dessen Projekt zwischenzeitlich gelÃ¶scht/ausgeblendet wurde, fÃ¤llt in diesem Lauf
  auf den globalen Standard zurÃ¼ck.

Siehe auch: [Import](importing.md) Â· [Automatisches Anlegen](auto-create.md).
