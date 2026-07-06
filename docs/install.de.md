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

Die Migrationen des Plugins betreffen ausschließlich seine **eigene** Tabelle
`kimai2_jira_token` – Kimais Kern-Tabellen werden nie verändert –, daher ist ein erneutes Ausführen
des Installers auf einer bestehenden Installation unbedenklich.

!!! warning "Das Upgrade von einer Release mit globaler Konfiguration ist ein harter Umstieg"
    Diese Release verschiebt die Jira-Konfiguration von der bisherigen Seite **System →
    Einstellungen → Jira** auf jeden **Kunden** und macht aus dem einzelnen benutzereigenen Token
    ein Token **pro Kunde**. Beim Upgrade werden bestehende Token verworfen und die alten globalen
    `jira.*`-Einstellungen entfernt. Nach dem Aktualisieren müssen Sie **Jira an jedem Kunden neu
    konfigurieren** (Server-URL, Auth-Modus, Import-Optionen – siehe [Einrichtung](configure.md))
    und jeder Benutzer muss **ein Token pro Kunde neu eingeben**.

!!! warning "`APP_SECRET` nicht ohne dedizierten Token-Schlüssel rotieren"
    Gespeicherte Jira-Token werden standardmäßig mit einem aus Kimais `APP_SECRET` abgeleiteten
    Schlüssel verschlüsselt. Ändert sich `APP_SECRET`, wird **jedes gespeicherte Token
    unentschlüsselbar** und jede Person muss ihr Token neu eingeben. Wenn Ihre Umgebung
    `APP_SECRET` rotiert, setzen Sie zuvor eine dedizierte Umgebungsvariable `JIRA_TOKEN_KEY` – der
    Vault leitet seinen Schlüssel dann daraus ab, sodass beide unabhängig rotieren können.

Weiter: [Einrichtung](configure.md).
