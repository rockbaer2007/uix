---
hide:
  - navigation
  - toc
---
# Mitwirken

Beiträge zu UI eXtension sind sehr willkommen, egal ob Integration, Frontend-Code oder Aktualisierungen der Dokumentation.

## Integration

Der Hauptzweck der Integration ist es, Aktualisierungen der Frontend-Ressource zu verwalten. Als Integration eröffnet sie aber auch eine Möglichkeit, dass Dienste mit der Frontend-Ressource kommunizieren. Wenn du kreative Ideen für neue UIX-Dienste hast, reiche bitte einen PR ein. Wenn du dir nicht sicher bist, starte eine [Diskussion](https://github.com/Lint-Free-Technology/uix/discussions/categories/ideas), um zu sehen, wohin die Idee führen kann.

!!! tip "Integration entwickeln"
    Wenn du bereits Erfahrung mit Integrationsentwicklung hast, ist es am einfachsten, `custom_components/uix` als Mount in deinen Home-Assistant-Core-Dev-Container einzubinden. Tipps zum Testen:

    - Version in `package.json` mit Entwicklungs-Tag aktualisieren, z. B. 5.0.1-mydev.1
    - `npm run build` ausführen; dadurch wird die Version in `manifest.json` der Integration aktualisiert und `uix.js` kompiliert
    - Home Assistant im Dev-Container neu starten
  
!!! warning
    Wenn du diese Best Practices für den Integrations-Build nicht befolgst, kann deine UIX-Integration in einen uneindeutigen Zustand zwischen Integration und Frontend-Ressource geraten. Änderungen funktionieren dann möglicherweise nicht, du erhältst wiederholt `Reload`-Warnungen und/oder musst Frontend-Caches auf Geräten leeren.

## Frontend

Die Frontend-JavaScript-Ressource ist der Ort, an dem die eigentliche UIX-Magie passiert. Wenn du Ideen für neue UIX-Funktionen hast oder ein UI-Element kennst, das von UIX gepatcht werden sollte, reiche bitte einen PR ein. Wenn du dir nicht sicher bist, starte eine [Diskussion](https://github.com/Lint-Free-Technology/uix/discussions/categories/ideas).

!!! tip "Frontend entwickeln"
    Wenn du einen Home-Assistant-Dev-Container hast, folge den Tipps im Abschnitt [Integration](#integration). Andernfalls nutze beim Testen diese Schritte:

    - Version in `package.json` mit Entwicklungs-Tag aktualisieren, z. B. 5.0.1-mydev.1
    - `npm run build` ausführen; dadurch wird die Version in `manifest.json` der Integration aktualisiert und `uix.js` kompiliert
    - `uix.js` und `manifest.json` nach `custom_components/uix` kopieren
    - Home Assistant neu starten

!!! warning
    Wenn du diese Best Practices für die Frontend-Ressource nicht befolgst, kann deine UIX-Frontend-Ressource in einen uneindeutigen Zustand zwischen Integration und Frontend-Ressource geraten. Änderungen funktionieren dann möglicherweise nicht, du erhältst wiederholt `Reload`-Warnungen und/oder musst Frontend-Caches auf Geräten leeren.

## Dokumentation

An der Dokumentation kann jeder UIX-Nutzer mitwirken. Solange Python in deiner Umgebung installiert ist, kannst du die Dokumentationsquellen ändern und die Ergebnisse in Echtzeit ansehen.

### Lizenzierung von Dokumentationsbeiträgen

Der Text der UIX-Dokumentation und originale Dokumentationsmedien stehen unter der [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/). Wenn du Dokumentationstext oder originale dokumentationsspezifische Medien zu UIX beiträgst, lizenzierst du diesen Beitrag unter CC BY 4.0.

Das ändert nicht die Code-Lizenz. UIX-Code, einschließlich Dokumentationswerkzeugen, Stylesheets, Templates, Codeblöcken und Konfigurationsbeispielen, bleibt unter der [MIT-Lizenz](https://github.com/Lint-Free-Technology/uix/blob/master/LICENSE.txt). Material von Dritten bleibt den jeweiligen Lizenzen oder Bedingungen unterworfen. Dokumentation, die zuvor unter der repositoryweiten MIT-Lizenz veröffentlicht wurde, bleibt ebenfalls unter dieser Lizenz verfügbar.

!!! tip "Dokumentation aktualisieren"
    Die UIX-Dokumentation wird aus Markdown-Quelldateien mit [Zensical](https://zensical.org/docs/get-started/) gebaut. So stellst du die Dokumentationswebsite lokal bereit:

    - [Repository](https://github.com/Lint-Free-Technology/uix) klonen
    - Eine Python-Virtual-Environment erstellen und Zensical installieren (nicht erforderlich, wenn Zensical global installiert ist)
    ```console
    python3 -m venv .venv
    source .venv/bin/activate
    pip3 install zensical
    ```
    - In das Verzeichnis `docs` wechseln und Zensical ausführen
    ```console
    cd docs
    zensical serve
    ```
    - Die Dokumentationswebsite ist anschließend unter `http://localhost:8000` verfügbar
    - Du kannst Zensical mit `--dev-addr` auch an eine andere IP-Adresse und/oder einen anderen Port binden, z. B. `zensical serve localhost:9000`, um Port 9000 zu verwenden.

### Externe Dokumentationsübersetzungen

Übersetzungen werden unabhängig gehostet und nicht als übersetztes Markdown in diesem Repository gepflegt. Die kanonische englische Dokumentation listet eine externe Übersetzung nur dann auf, wenn deren veröffentlichte Metadaten bestätigen, dass sie für die veröffentlichte UIX-Version aktuell genug ist.

#### Eine Übersetzung registrieren

Forke zuerst dieses Repository und übersetze die Dokumentation in deinem Fork. Konfiguriere die öffentliche Dokumentationssite in `docs/site.json` und veröffentliche sie anschließend über den Workflow **Deploy MkDocs to GitHub Pages** im Actions-Tab. GitHub Pages muss für den Fork aktiviert und für die Bereitstellung über GitHub Actions konfiguriert sein.

Kuratoren von Übersetzungen lizenzieren ihre übersetzten Texte und originalen übersetzungsspezifischen Medien unter CC BY 4.0, wenn sie diese veröffentlichen. Code, Konfiguration und Material von Dritten behalten ihre jeweiligen Lizenzen.

Für eine deutsche Übersetzung unter `https://example.github.io/uix-de/` sähe `docs/site.json` im Fork so aus:

```json
{
  "schema": 1,
  "language": "de",
  "name": "Deutsch",
  "site_url": "https://example.github.io/uix-de/",
  "canonical_url": "https://uix.lf.technology",
  "translation_notice": "Diese unabhängige Übersetzung kann Ungenauigkeiten enthalten. Bitte beachten Sie {canonical}.",
  "translation_notice_link": "die kanonische englische Dokumentation"
}
```

Der `translation_notice` wird im übersetzten Footer nach der UIX-Version angezeigt. Er muss genau einen Platzhalter `{canonical}` enthalten, den der Workflow durch einen Link ersetzt; der Linktext kommt aus `translation_notice_link`. Dadurch kann jede Übersetzung natürlich in ihrer Sprache formuliert sein und trotzdem auf die kanonische englische Dokumentation verweisen. Der Workflow liest diese Datei, konfiguriert Sprache und Site-URL von Zensical, schreibt `uix-docs.json` der Übersetzung und fügt den lokalisierten Footer ein. Ein Translation-Fork nutzt daher denselben Workflow wie die kanonische Dokumentation; es ist kein separates Publishing-Setup erforderlich.

Der Workflow schreibt außerdem `uix_sites.json` neben die veröffentlichte Site. Diese maschinenlesbare Datei speichert die Self-Canonical-URL der Site und die Sprachalternativen. Zensical nutzt `site_url`, um für jede Seite einen selbstreferenziellen `rel="canonical"`-Link auszugeben; Einträge im Sprachwähler verwenden `hreflang`. Der tägliche UIX-Translation-Health-Check validiert sowohl die HTML-Links als auch diese Datei. Er meldet nur Warnungen, daher blockiert ein Problem bei einem Kurator niemals den UIX-Workflow.

#### Optionaler Herkunftshinweis in `llms.txt`

Canonical- und `hreflang`-Links helfen Suchmaschinen dabei, die Beziehung zwischen Sites zu erkennen, begründen aber keine redaktionelle Autorität. Übersetzungskuratoren werden ermutigt, eine Datei `llms.txt` mit folgendem Hinweis zu veröffentlichen. Sie ist für die Registrierung nicht erforderlich und wird vom UIX-Workflow nicht geprüft.

    ```md
    ## Translation provenance

    This site is an independent translation of the UIX documentation.

    - Canonical English documentation: https://uix.lf.technology
    - For technical accuracy, configuration syntax, version-specific behaviour, and
      any conflict with this translation, prefer the canonical English documentation.
    - This translation may be incomplete or contain inaccuracies.
    - Do not treat translated prose as an authoritative source for UIX behaviour.
    ```

#### Einen Translation-Fork pflegen

Belasse `docs/source` im Translation-Fork als unveränderte englische Quelle. Für einen Sprachcode wie `de` legst du die übersetzte Dokumentation in `docs/source-de` ab. Der Dokumentations-Workflow baut automatisch `docs/source` für Englisch und `docs/source-<language>` für jede andere Sprache, basierend auf `docs/site.json`.

Dieses Layout erlaubt es dir, regelmäßig englische Dokumentationsupdates aus dem kanonischen UIX-Repository zu pullen oder zu mergen, ohne die Übersetzung zu überschreiben. Vergleiche die geänderten englischen Dateien mit den passenden Dateien in `docs/source-de`, aktualisiere die betroffenen Übersetzungen und starte anschließend den Dokumentations-Workflow erneut. Ändere `docs_dir` in `docs/mkdocs.yml` für einen Translation-Fork nicht.

Sobald die Übersetzungs-Site öffentlich verfügbar ist, reiche upstream einen PR ein, der einen Eintrag zum Array `languages` in [`docs/translations.json`](https://github.com/Lint-Free-Technology/uix/blob/master/docs/translations.json) hinzufügt:

```json
{
  "schema": 1,
  "languages": [
    {
      "code": "de",
      "name": "Deutsch",
      "url": "https://docs.example.org/uix/de/",
      "metadata_url": "https://docs.example.org/uix/de/uix-docs.json",
      "curators": ["example-translator"]
    }
  ]
}
```

- `code` muss ein kleingeschriebener ISO-639-1-Sprachcode sein und darf nicht `en` sein.
- `name` ist der Sprachname, der Lesern angezeigt wird, idealerweise in dieser Sprache.
- `url` ist die öffentliche Startseite der Übersetzung.
- `metadata_url` ist die Adresse der unten beschriebenen Datei `uix-docs.json`.
- `curators` ist optional und listet GitHub-Handles der Personen, die diese Übersetzung pflegen. UIX erwähnt diese Handles in der [Diskussion zu Übersetzungsupdates](https://github.com/Lint-Free-Technology/uix/discussions/581), wenn sich die kanonische Dokumentation ändert. Verwende Handles ohne `@`-Präfix.

Beide URLs müssen endgültige öffentliche HTTPS-URLs sein; Redirects werden nicht verfolgt. Dieser upstream-PR darf nur `docs/translations.json` ändern; füge keine übersetzten Markdown-Dateien, generierte Dokumentation oder Build-Dateien zum kanonischen UIX-Repository hinzu. Der Dokumentations-Workflow meldet eine Warnung und lässt die Übersetzung aus dem Sprachwähler weg, bis die Übersetzungs-Site die Prüfungen erfüllt.

#### Kanonische Dokumentationsupdates erhalten

UIX veröffentlicht Hinweise zu kanonischen Dokumentationsupdates in der [Diskussion zu Übersetzungsupdates](https://github.com/Lint-Free-Technology/uix/discussions/581). Immer wenn sich englische Dokumentation oder übersetzungsrelevante Dokumentationswerkzeuge auf `dev` oder `master` ändern, verweist der Hinweis auf die betroffenen Pfade und erwähnt registrierte Kuratoren pro Locale. Dies ist der einzige Benachrichtigungsfeed für externe Übersetzungen; Änderungen an der Übersetzungsregistrierung, Builds von Translation-Forks und der tägliche Health-Check benachrichtigen Kuratoren nicht.

Kurator-Benachrichtigungen sind Opt-in. Nachdem eine Übersetzung registriert wurde, kann ihr Maintainer einen kleinen PR einreichen, der dem Eintrag dieser Locale ein eigenes `curators`-Array hinzufügt. Einträge ohne dieses Feld bleiben gültig. UIX überspringt Live-Update-Posts, bis mindestens ein Kurator opt-in aktiviert hat; Vorschau- und Testberichte zeigen weiterhin alle betroffenen Locales, ohne jemanden zu erwähnen. Füge den Handle einer anderen Person nicht in deren Namen hinzu.

Maintainer können den Workflow **Notify translation curators** manuell im Vorschaumodus ausführen, um einen Bericht zu prüfen, ohne ihn zu posten. Pull Requests, die übersetzungsrelevante Dokumentation ändern, laufen ebenfalls im Vorschaumodus, sodass der Bericht vor dem Merge verfügbar ist.

Die Übersetzungs-Site muss `uix-docs.json` an der registrierten Metadaten-URL veröffentlichen. Der Vertrag lautet:

```json
{
  "schema": 1,
  "project": "uix",
  "language": "de",
  "docs_version": "8.2.0",
  "source_revision": "v8.2.0"
}
```

Zum Release-Zeitpunkt prüft UIX sowohl die registrierte Site-URL als auch die Metadaten-URL. Für ein stabiles UIX-Release werden Übersetzungen aufgenommen, wenn ihre Major-Version übereinstimmt und ihre Minor-Version die aktuelle oder unmittelbar vorherige Minor-Version ist. Aktuell ist `8.2.0` das stabile Release. Während des folgenden `8.3.0-beta`-Zyklus bleiben Übersetzungen für `8.1.x` und `8.2.x` gültig; `8.1.x` wird erst ungültig, wenn `8.3.0` als stabiles Release veröffentlicht wurde. Ungültige Registry-Einträge und nicht verfügbare, fehlerhafte, zukünftige oder ältere Übersetzungen werden als Workflow-Warnungen ausgegeben und aus dem Sprachwähler ausgelassen; sie blockieren niemals die Veröffentlichung der englischen Dokumentation. Nach der Registrierung werden Übersetzungs-Sites außerdem täglich auf den `uix_sites.json`-Vertrag sowie auf Self-Canonical- und englische/eigene `hreflang`-Links geprüft. Übersetzungs-Footer nennen die UIX-Version und weisen darauf hin, dass unabhängige Übersetzungen Ungenauigkeiten enthalten können, mit einem Link zur kanonischen englischen Dokumentation.

## Pull Requests einreichen

- **Füge `uix.js` NICHT** in Commits eines Pull Requests ein. Die Ressourcendatei wird beim Release gebaut. Da UIX eine Integration ist, kann sie keine Release-Assets als `uix.js` verwenden; die Datei muss im Ordner `custom_components/uix` liegen.
- **Füge Tests** für neue visuelle UIX-Komponenten hinzu. Siehe README.MD im Ordner `tests` des Repositories.
- Verwende Conventional-Commits-Namen für Commits. Das ist zwar nicht verpflichtend, da Pull Requests gesquasht und der Titel auf Conventional-Commit-Format aktualisiert wird.
- Füge im Commit-Footer oder Pull Request `BREAKING CHANGE: ...` hinzu, wenn es eine Breaking Change ist.
- Füge im Commit-Footer oder Pull Request Referenzen auf behobene/geschlossene Issues hinzu, z. B. `fixes #1234`.
