# Installation

Dies ist ein kostenpflichtiges Plugin, das als Release-ZIP ausgeliefert wird. Entpacken Sie es in
das `var/plugins/`-Verzeichnis Ihrer Kimai-Installation und führen Sie den Installer aus:

```bash
cd /path/to/kimai
unzip JiraBundle-vX.Y.Z.zip -d var/plugins/   # entpackt nach var/plugins/JiraBundle/
bin/console cache:clear
bin/console kimai:bundle:jira:install         # legt die plugin-eigene Tabelle an
bin/console cache:clear
```

`kimai:plugins` sollte nun **JiraBundle** auflisten.

## Lizenzschlüssel eingeben

JiraBundle ist ein kostenpflichtiges Plugin – die Jira-Funktionen bleiben aus, bis ein gültiger
Schlüssel gesetzt ist. Fügen Sie den beim Kauf erhaltenen Schlüssel unter **System → Einstellungen →
Jira** ein oder setzen Sie die Umgebungsvariable `JIRA_LICENSE_KEY` (die Vorrang hat). Details, das
Kulanzfenster und der `kimai:jira:sync`-Heartbeat finden Sie unter [Lizenz](licensing.md).

## Voraussetzungen

- **Kimai** ≥ 2.21.0
- **PHP** ≥ 8.2
- **ext-sodium** – für den verschlüsselten Token-Speicher (Kimais dokumentierte PHP-Einrichtung
  hat es normalerweise bereits).
- **ext-xsl** – wird von Kimai selbst benötigt und ist zum Rendern jeglicher E-Mail erforderlich;
  ohne die Erweiterung schlägt der Versand der Benachrichtigungs-E-Mails still fehl.
- Ein funktionierendes `MAILER_DSN` und ein Cron-Eintrag – siehe [Einrichtung](configure.md) – für
  die Benachrichtigungs-E-Mails und den Hintergrund-Abgleich/Importer.

Führen Sie diese Befehle im Kimai-Stammverzeichnis aus, um alle drei vor der Installation zu
prüfen:

```bash
bin/console --version                 # Kimai-Version – muss ≥ 2.21.0 sein
php -v | head -n1                     # PHP-Version – muss ≥ 8.2 sein
php -m | grep -qi sodium && echo "ext-sodium: OK" || echo "ext-sodium: FEHLT — php-sodium installieren"
```

Oder als Pass/Fail-Block:

```bash
php -r 'exit(version_compare(PHP_VERSION,"8.2.0",">=")?0:1);' \
  && echo "PHP $(php -r 'echo PHP_VERSION;'): OK" || echo "PHP zu alt (≥ 8.2 nötig)"
php -r 'exit(extension_loaded("sodium")?0:1);' \
  && echo "ext-sodium: OK" || echo "ext-sodium: FEHLT"
```

`ext-sodium` ist zwingend erforderlich – ohne die Erweiterung kann das Plugin keine Token
speichern und der Installer verweigert die Ausführung.

## Aktualisieren

Legen Sie ein neueres ZIP über dasselbe Verzeichnis `var/plugins/JiraBundle`, führen Sie dann den
Installer erneut aus (er wendet neue Migrationen an) und leeren Sie den Cache:

```bash
bin/console kimai:bundle:jira:install
bin/console cache:clear
```

Die Migrationen des Plugins legen seine **eigene** Tabelle `kimai2_jira_token` an und verwalten sie
und tragen zwei plugin-eigene Zeilen in Kimais gemeinsame Tabelle `kimai2_configuration` ein –
`jira.token_key` (den Schlüssel zur Token-Verschlüsselung) und `jira.license_state` (den Anker der
Lizenz-Uhr). Beide werden nur eingetragen, wenn sie noch fehlen, daher ist ein erneutes Ausführen
des Installers auf einer bestehenden Installation unbedenklich und überschreibt keinen der beiden
Werte.

!!! warning "Das Upgrade von einer Release mit globaler Konfiguration ist ein harter Umstieg"
    Diese Release verschiebt die Jira-Konfiguration von der bisherigen Seite **System →
    Einstellungen → Jira** auf jeden **Kunden** und macht aus dem einzelnen benutzereigenen Token
    ein Token **pro Kunde**. Beim Upgrade werden bestehende Token verworfen und die alten globalen
    `jira.*`-Einstellungen entfernt. Nach dem Aktualisieren müssen Sie **Jira an jedem Kunden neu
    konfigurieren** (Server-URL, Auth-Modus, Import-Optionen – siehe [Einrichtung](configure.md))
    und jeder Benutzer muss **ein Token pro Kunde neu eingeben**.

!!! warning "Der Token-Schlüssel liegt in der Datenbank – sichern Sie ihn als Einheit"
    Gespeicherte Jira-Token werden mit einem Schlüssel verschlüsselt, der einmalig bei der
    Installation erzeugt und in Ihrer Kimai-Datenbank abgelegt wird (Zeile `jira.token_key` in
    `kimai2_configuration`). Da der Schlüssel mit der Datenbank wandert, trägt eine normale
    Datenbank-Sicherung oder -Migration ihn mit – stellen Sie die Datenbank wieder her, und die
    gespeicherten Token lassen sich wieder entschlüsseln. Eine Rotation von Kimais `APP_SECRET`
    betrifft gespeicherte Token **nicht mehr**. Verloren gehen die Token nur, wenn diese
    Datenbank-Zeile verloren geht: Spielen Sie die Anwendungsdateien auf eine **leere** oder andere
    Datenbank zurück oder löschen Sie die Zeile, wird jedes gespeicherte Token unentschlüsselbar und
    jede Person muss ihr Token neu eingeben.

!!! note "Fortgeschritten: den Schlüssel mit `JIRA_TOKEN_KEY` festlegen"
    Normalerweise müssen Sie das nie anfassen. `JIRA_TOKEN_KEY` ist ein **Override** – ist die
    Variable gesetzt, verwendet der Vault sie anstelle des in der Datenbank gespeicherten Schlüssels,
    für Umgebungen, die den Schlüssel lieber zusammen mit ihren übrigen Geheimnissen verwalten. Auf
    einer Installation, die **bereits Token gespeichert hat**, setzen Sie nicht einfach einen neuen
    `JIRA_TOKEN_KEY`: Diese Token wurden mit dem Datenbank-Schlüssel verschlüsselt, und das Festlegen
    eines anderen Schlüssels macht sie allesamt unbrauchbar. Um sicher auf einen festgelegten
    Schlüssel umzustellen, kopieren Sie den bestehenden Wert aus der Zeile `jira.token_key` und
    setzen `JIRA_TOKEN_KEY` auf genau diesen Wert; prüfen Sie dann, dass eine Person ihr
    gespeichertes Token weiterhin laden kann, bevor Sie die Zeile entfernen. Ein `JIRA_TOKEN_KEY` von
    Grund auf ist nur für eine brandneue Installation ohne bereits gespeicherte Token gedacht.

Weiter: [Einrichtung](configure.md).
