# Lizenz

JiraBundle ist ein kostenpflichtiges Plugin. Die Kimai-Zeiterfassung läuft stets ohne Lizenz, doch
die **eigenen Jira-Funktionen des Plugins – Worklog-Sync, Import und die Live-Vorgangssuche – laufen
nur mit einem gültigen Lizenzschlüssel.** Diese Seite beschreibt, wie Sie die Lizenz kaufen, woher
der Schlüssel kommt, die beiden Wege, ihn zu setzen, und was genau geschieht, wenn ein Abonnement
ausläuft oder der Schlüssel nicht geprüft werden kann.

## Lizenz kaufen

JiraBundle wird als **Abonnement für 49 €/Jahr** verkauft. Kaufen Sie es hier:
**[JiraBundle kaufen – 49 €/Jahr](https://buy.stripe.com/00wbJ32Bhfyo0AQevdefC01)**.

Nach dem Bezahlvorgang erhalten Sie den signierten Lizenzschlüssel per E-Mail. Fügen Sie ihn in
Kimai unter **System → Einstellungen → Jira** ein – [Den Schlüssel setzen](#den-schlussel-setzen)
weiter unten beschreibt beide Wege, ihn bereitzustellen, und welcher Vorrang hat.

JiraBundle ist auch im
[Kimai-Marktplatz](https://www.kimai.org/en/store/jira-sync.html) gelistet; Sie können es dort oder
über den Bezahllink oben kaufen.

## Woher der Schlüssel kommt

Sie erhalten den Lizenzschlüssel per E-Mail beim Kauf des Abonnements – dieselbe Auslieferung wie
das Release-ZIP. Es handelt sich um ein signiertes Token (`v1.<payload>.<signature>`); fügen Sie es
unverändert ein, einschließlich des Präfixes `v1.`.

Auch der Beginn eines Testzeitraums sendet einen Schlüssel per E-Mail, gültig bis zum Ende des
Testzeitraums. Nach jeder erfolgreichen Verlängerungszahlung kommt ein neuer Schlüssel per E-Mail,
und die tägliche Lizenzprüfung ruft automatisch einen verlängerten Schlüssel ab, wenn sie den
Lizenzdienst erreicht (siehe [Verlängerung und Ablauf](#verlangerung-und-ablauf)). Eine
fehlgeschlagene Verlängerungszahlung sendet keinen Schlüssel.

## Den Schlüssel setzen

Es gibt zwei Wege, ihn bereitzustellen, und sie haben eine **bewusste Rangfolge**:

1. **Umgebungsvariable `JIRA_LICENSE_KEY`** – für Container- und automatisierte Deployments. **Diese
gewinnt.** Ist sie gesetzt und nicht leer, verwendet das Plugin sie und ignoriert den Wert aus der
Systemkonfiguration. 2. **Systemkonfiguration** – öffnen Sie als Administrator **System →
Einstellungen → Jira** und fügen Sie den Schlüssel in das Feld **Lizenzschlüssel** ein. Wird
verwendet, sobald `JIRA_LICENSE_KEY` nicht gesetzt oder leer ist.

!!! note "Die Umgebungsvariable überschreibt das Einstellungsfeld"
    Ist `JIRA_LICENSE_KEY` gesetzt, hat das Bearbeiten des Schlüssels unter **System → Einstellungen
    → Jira** keine Wirkung – die Umgebungsvariable wird zuerst gelesen und beendet die Suche vorzeitig.
    Verwalten Sie den Schlüssel bei einem containerisierten Deployment über die Umgebungsvariable und
    lassen Sie das Einstellungsfeld leer, um Verwechslungen zu vermeiden.

Ein Verlängerungsschlüssel, den das Plugin selbst abgerufen hat, hat Vorrang vor beiden, solange der
konfigurierte Schlüssel eine gültige Signatur trägt, auch nach seinem Ablauf. Ein Schlüssel, den Sie
für dasselbe Abonnement mit späterem Ablaufdatum einfügen, hat Vorrang vor dem abgerufenen. Siehe
[Verlängerung und Ablauf](#verlangerung-und-ablauf).

## Offline-Verifizierung

Der Schlüssel wird **vollständig offline** verifiziert. Er trägt eine Ed25519-Signatur, die das
Plugin gegen einen eingebetteten öffentlichen Schlüssel prüft – für den *Betrieb* des Plugins ist
kein Aufruf eines Servers nötig. Eine air-gapped Installation mit einem gültigen, nicht abgelaufenen
Schlüssel funktioniert vollständig ohne ausgehende Netzwerkverbindung (vorbehaltlich der
Offline-Veraltungs-Uhr weiter unten).

## Das Kulanzfenster: was aussetzt und wann

Läuft eine Lizenz ab, wird sie widerrufen oder veraltet sie (siehe unten), werden die
Jira-Funktionen **nicht** sofort abgeschaltet. Es gibt ein **14-tägiges Kulanzfenster**: die
Funktionen laufen weiter, und Kimai zeigt ein eskalierendes Banner mit einem Countdown der
verbleibenden Tage. Das Banner nennt die Ursache:

**Der Schlüssel ist abgelaufen.** Sein Ablaufdatum ist überschritten, und noch kein verlängerter
Schlüssel ist eingetroffen:

> Ihr Jira-Lizenzschlüssel ist am *Datum* abgelaufen. Die Jira-Synchronisierung läuft noch *N*
> Tag(e). Solange die tägliche Lizenzprüfung (kimai:jira:sync) läuft, wird der
> Verlängerungsschlüssel nach der Zahlung automatisch abgerufen. Andernfalls fügen Sie den Schlüssel
> aus der Verlängerungs-E-Mail in den Systemeinstellungen ein.

**Das Abonnement ist inaktiv.** Der Lizenzdienst meldet eine fehlgeschlagene Zahlung oder ein
gekündigtes Abonnement:

> Der Lizenzdienst meldet dieses Jira-Abonnement als inaktiv (Zahlung fehlgeschlagen oder Abonnement
> gekündigt). Die Jira-Synchronisierung läuft noch *N* Tag(e). Nach erfolgreicher Zahlung stellt die
> nächste tägliche Lizenzprüfung es wieder her. Details in den Systemeinstellungen.

**Die Lizenzprüfung ist veraltet.** Kimai hat den Lizenzdienst innerhalb des
[Offline-Veraltungs-Fensters](#die-offline-veraltungs-uhr-air-gapped-installationen) nicht erreicht:

> Kimai hat den Jira-Lizenzdienst seit dem *Datum* nicht erreicht. Die Jira-Synchronisierung läuft
> noch *N* Tag(e). Prüfen Sie, ob kimai:jira:sync läuft und den Lizenz-Host erreichen kann.

Jedes Banner endet mit der Adresse der Einstellungsseite.

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
als „unbekannt“ verbucht und deaktiviert eine funktionierende Instanz nie. Dieselbe Prüfung ruft
Verlängerungsschlüssel ab (siehe [Verlängerung und Ablauf](#verlangerung-und-ablauf)).

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
    vom Lizenzdienst entkoppelt zu laufen. Das Plugin kann auch keine Verlängerungsschlüssel abrufen;
    fügen Sie daher bei jeder Verlängerung den Schlüssel aus der E-Mail ein.

## Verlängerung und Ablauf

- **Die Verlängerung läuft automatisch**, solange `kimai:jira:sync` läuft und den Lizenzdienst
  erreicht. Nach erfolgreicher Verlängerungszahlung ruft die tägliche Lizenzprüfung den neuen
  Schlüssel vom Lizenzdienst ab, prüft ihn (Signatur, gleicher Kunde, späteres Ablaufdatum),
  speichert ihn und verwendet ihn fortan statt des konfigurierten Schlüssels. Sie müssen nichts
  einfügen. - Der automatische Abruf setzt voraus, dass der Lizenz-Host `https` verwendet oder
  `JIRA_ALLOW_INSECURE_URL` gesetzt ist. Der Standard-Host verwendet `https`. Das betrifft Sie daher
  nur, wenn Sie `JIRA_LICENSE_HOST` überschreiben. `JIRA_ALLOW_INSECURE_URL` deaktiviert auch die
  Prüfung der Jira-Server-URL. Über einfaches `http` läuft die tägliche Prüfung weiterhin als
  Widerrufsprüfung, sendet aber den Schlüssel nicht und ruft keinen Verlängerungsschlüssel ab. -
  **Fügen Sie den Schlüssel aus der Verlängerungs-E-Mail** unter **System → Einstellungen → Jira**
  ein oder aktualisieren Sie die Umgebungsvariable `JIRA_LICENSE_KEY`, wenn der Cron nicht läuft
  oder den Lizenzdienst nicht erreicht (air-gapped mit `JIRA_LICENSE_OFFLINE_GRACE_DAYS=0` oder
  hinter einer Firewall), der Lizenz-Host einfaches `http` verwendet oder der alte Schlüssel vor
  mehr als 30 Tagen abgelaufen ist. Ein von Hand eingefügter neuerer Schlüssel hat Vorrang vor einem
  abgerufenen. Beim nächsten Lauf von `kimai:jira:sync` prüft der Heartbeat das Abonnement erneut
  und aktiviert die Funktionen wieder; in der Warteschlange stehende Worklogs werden dann
  nachgetragen. - Eine **fehlgeschlagene Verlängerungszahlung** sendet keinen Schlüssel. Der
  Lizenzdienst meldet das Abonnement als inaktiv, und das Kulanzfenster beginnt; nach erfolgreicher
  Zahlung stellt die nächste tägliche Prüfung es wieder her. - Wenn Sie den konfigurierten Schlüssel
  leeren, verliert die Installation weiterhin ihre Lizenz. Ein abgerufener Schlüssel ersetzt nie
  einen konfigurierten Schlüssel, dessen Signatur ungültig ist. - Ein **anhaltender Ausfall des
  Lizenzdienstes selbst** (~44 Tage: 30 Veraltung + 14 Kulanz) wird schließlich die Jira-Funktionen
  auf ansonsten gesunden, online betriebenen, bezahlten Installationen deaktivieren. Kimai und Ihre
  Daten bleiben unberührt, und das Plugin **heilt sich selbst** beim nächsten erfolgreichen Kontakt.
  Falls dieser Kompromiss für Sie relevant ist – air-gapped, bewusst offline oder Sie möchten sich
  vollständig von der Verfügbarkeit des Anbieters entkoppeln – setzen Sie
  `JIRA_LICENSE_OFFLINE_GRACE_DAYS=0`.

Wenn Jira-Funktionen unerwartet aussetzen, siehe [Fehlerbehebung → Jira-Funktionen funktionieren
nicht mehr](features/troubleshooting.md#jira-funktionen-funktionieren-nicht-mehr).
