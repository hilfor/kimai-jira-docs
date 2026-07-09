# Lizenz

JiraBundle ist ein kostenpflichtiges Plugin. Die Kimai-Zeiterfassung läuft stets ohne Lizenz, doch
die **eigenen Jira-Funktionen des Plugins – Worklog-Sync, Import und die Live-Vorgangssuche – laufen
nur mit einem gültigen Lizenzschlüssel.** Diese Seite beschreibt, woher der Schlüssel kommt, die
beiden Wege, ihn zu setzen, und was genau geschieht, wenn ein Abonnement ausläuft oder der Schlüssel
nicht geprüft werden kann.

## Woher der Schlüssel kommt

Sie erhalten den Lizenzschlüssel per E-Mail beim Kauf des Abonnements – dieselbe Auslieferung wie
das Release-ZIP. Es handelt sich um ein signiertes Token (`v1.<payload>.<signature>`); fügen Sie es
unverändert ein, einschließlich des Präfixes `v1.`.

## Den Schlüssel setzen

Es gibt zwei Wege, ihn bereitzustellen, und sie haben eine **bewusste Rangfolge**:

1. **Umgebungsvariable `JIRA_LICENSE_KEY`** – für Container- und automatisierte Deployments. **Diese
   gewinnt.** Ist sie gesetzt und nicht leer, verwendet das Plugin sie und ignoriert den Wert aus der
   Systemkonfiguration.
2. **Systemkonfiguration** – öffnen Sie als Administrator **System → Einstellungen → Jira** und fügen
   Sie den Schlüssel in das Feld **Lizenzschlüssel** ein. Wird verwendet, sobald `JIRA_LICENSE_KEY`
   nicht gesetzt oder leer ist.

!!! note "Die Umgebungsvariable überschreibt das Einstellungsfeld"
    Ist `JIRA_LICENSE_KEY` gesetzt, hat das Bearbeiten des Schlüssels unter **System → Einstellungen
    → Jira** keine Wirkung – die Umgebungsvariable wird zuerst gelesen und beendet die Suche vorzeitig.
    Verwalten Sie den Schlüssel bei einem containerisierten Deployment über die Umgebungsvariable und
    lassen Sie das Einstellungsfeld leer, um Verwechslungen zu vermeiden.

## Offline-Verifizierung

Der Schlüssel wird **vollständig offline** verifiziert. Er trägt eine Ed25519-Signatur, die das
Plugin gegen einen eingebetteten öffentlichen Schlüssel prüft – für den *Betrieb* des Plugins ist
kein Aufruf eines Servers nötig. Eine air-gapped Installation mit einem gültigen, nicht abgelaufenen
Schlüssel funktioniert vollständig ohne ausgehende Netzwerkverbindung (vorbehaltlich der
Offline-Veraltungs-Uhr weiter unten).

## Das Kulanzfenster: was aussetzt und wann

Läuft eine Lizenz ab, wird sie widerrufen oder veraltet sie (siehe unten), werden die Jira-Funktionen
**nicht** sofort abgeschaltet. Es gibt ein **14-tägiges Kulanzfenster**: die Funktionen laufen
weiter, und Kimai zeigt ein eskalierendes Banner mit einem Countdown der verbleibenden Tage. Das
Banner lautet:

> Jira subscription lapsed — sync keeps working for *N* more day(s), then Jira features are
> disabled. Renew and update the license key under System settings.

Sind die 14 Tage verstrichen, werden die Jira-Funktionen des Plugins **abgeschaltet**:

- **Worklog-Sync** (inline beim Speichern eines Zeiteintrags sowie der Abgleich `kimai:jira:sync`)
  pausiert. Ausstehende Worklogs gehen **nicht verloren** – sie bleiben in der Warteschlange und
  werden nachgetragen, sobald wieder ein gültiger Schlüssel aktiv ist.
- **Import** (`kimai:jira:import`) ist deaktiviert und beendet sich ohne zu importieren.
- **Live-Vorgangssuche** (die beratende Vorgangsschlüssel-Prüfung im Zeiteintrag-Formular)
  verstummt – sie liefert nichts zurück, genau wie eine nicht erreichbare Jira, und blockiert nie
  das Speichern eines Zeiteintrags.

**Kimai-Core, die Zeiterfassung und alle vorhandenen Daten werden nie berührt.** Diese Sperre
entscheidet ausschließlich darüber, ob das plugin-eigene Jira-Verhalten läuft.

Ist überhaupt kein Schlüssel konfiguriert (statt eines abgelaufenen), sind dieselben Funktionen aus,
und das Banner fordert Sie stattdessen auf, den beim Kauf erhaltenen Schlüssel einzufügen.

## Der tägliche Widerrufs-Heartbeat läuft über `kimai:jira:sync`

Die Verifizierung ist offline, doch das Plugin braucht dennoch einen Weg, ein **widerrufenes**
Abonnement zu bemerken (eine Rückerstattung oder Rückbuchung). Dazu dient ein einmal täglicher
**Widerrufs-Heartbeat**, der über den Cron `kimai:jira:sync` läuft. Der Heartbeat ist
**fail-open**: nur eine ausdrückliche „widerrufen/inaktiv“-Antwort des Lizenzdienstes startet die
Kulanz-Uhr. Ein nicht erreichbarer Endpunkt, eine Nicht-2xx-Antwort oder nicht parsebares JSON wird
als „unbekannt“ verbucht und deaktiviert eine funktionierende Instanz nie.

!!! warning "Planen Sie `kimai:jira:sync` ein, sonst degradiert Ihre bezahlte Installation von selbst"
    Der Cron `kimai:jira:sync` ist das **einzige**, das die Offline-Veraltungs-Uhr (unten)
    zurücksetzt. Die Live-Funktionen des Plugins – Inline-Worklog-Sync beim Speichern eines
    Zeiteintrags, Vorgangssuche – laufen *ohne* den Cron, daher bleibt er leicht ungeplant. Doch
    dann degradiert eine bezahlte, online betriebene Installation ihre Jira-Funktionen nach rund
    **44 Tagen**, berechnet als **30 Tage Offline-Veraltung + 14 Tage Kulanz** – weil sie sich nie
    meldet. **Planen Sie den Cron ein** (siehe [Einrichtung → Cron](configure.md)) oder setzen Sie
    `JIRA_LICENSE_OFFLINE_GRACE_DAYS=0`, um die Offline-Uhr vollständig zu deaktivieren.

## Die Offline-Veraltungs-Uhr (air-gapped Installationen)

Fail-open allein würde einer Kopie, die den Lizenz-Host dauerhaft blockiert, erlauben, den Widerruf
ewig zu umgehen. Um das zu begrenzen, führt das Plugin eine **Offline-Veraltungs-Uhr**: nach **30
Tagen** ohne erfolgreichen Kontakt mit dem Lizenzdienst startet die Veraltung dasselbe
14-tägige Kulanz-dann-Abschaltung-Fenster. Daher stammt die ~44-Tage-Angabe – **30 (Veraltung) + 14
(Kulanz)**.

- Die Uhr ist daran verankert, wann *diese Installation* sich zum ersten Mal gemeldet hat (ihr
  Erst-Kontakt-Zeitpunkt), **nicht** daran, wann der Schlüssel ausgestellt wurde – sodass die
  Wiederherstellung aus einer Sicherung mit einem älteren Schlüssel Sie nie bereits abgelaufen
  starten lässt.
- **Jeder erfolgreiche tägliche Heartbeat setzt sie zurück**, sodass eine normal verbundene
  Installation sie nie auslöst.
- Das Fenster wird über **`JIRA_LICENSE_OFFLINE_GRACE_DAYS`** (Standard `30`) gesetzt:
    - **erhöhen Sie es** für nur zeitweise verbundene Standorte, oder
    - setzen Sie es auf **`0`, um die Offline-Uhr vollständig zu deaktivieren** für eine legitim
      air-gapped Installation.

!!! note "Air-gapped Installationen"
    Setzen Sie auf einer bewusst offline betriebenen Installation `JIRA_LICENSE_OFFLINE_GRACE_DAYS=0`.
    Der signierte Schlüssel bleibt für seine gesamte Laufzeit maßgeblich, und die Offline-Uhr greift
    nie. Sie erhalten dann keine Widerrufs-Aktualisierungen – das ist der Preis dafür, vollständig
    vom Lizenzdienst entkoppelt zu laufen.

## Verlängerung und Ablauf

- **Verlängern** Sie, indem Sie den Schlüssel aktualisieren – fügen Sie den neuen Schlüssel unter
  **System → Einstellungen → Jira** ein oder aktualisieren Sie die Umgebungsvariable
  `JIRA_LICENSE_KEY`. Beim nächsten Lauf von `kimai:jira:sync` prüft der Heartbeat das Abonnement
  erneut und aktiviert die Funktionen wieder; in der Warteschlange stehende Worklogs werden dann
  nachgetragen.
- Ein **anhaltender Ausfall des Lizenzdienstes selbst** (~44 Tage: 30 Veraltung + 14 Kulanz) wird
  schließlich die Jira-Funktionen auf ansonsten gesunden, online betriebenen, bezahlten
  Installationen deaktivieren. Kimai und Ihre Daten bleiben unberührt, und das Plugin **heilt sich
  selbst** beim nächsten erfolgreichen Kontakt. Falls dieser Kompromiss für Sie relevant ist –
  air-gapped, bewusst offline oder Sie möchten sich vollständig von der Verfügbarkeit des Anbieters
  entkoppeln – setzen Sie `JIRA_LICENSE_OFFLINE_GRACE_DAYS=0`.

Wenn Jira-Funktionen unerwartet aussetzen, siehe
[Fehlerbehebung → Jira-Funktionen funktionieren nicht mehr](features/troubleshooting.md#jira-funktionen-funktionieren-nicht-mehr).
