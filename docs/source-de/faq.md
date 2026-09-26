---
description: Antworten auf häufige Fragen zu UI eXtension.
hide:
  - toc
  - navigation
---
# Häufige Fragen

## Wie migriere ich am besten von Card-mod?

- Deinstalliere Card-mod.
- Wenn du `extra_module_url` für die Card-mod-Ressource verwendest, entferne den Eintrag und starte Home Assistant neu.
- Folge anschließend dem [Schnellstart für UI eXtension](./quick-start.md).

!!! tip "UI eXtension als Dienst hinzufügen"
    UI eXtension ist eine Integration. Nachdem du sie über HACS heruntergeladen hast, musst du sie als Dienst hinzufügen. Achte darauf, den Schritt **UI eXtension-Dienst hinzufügen** nicht zu überspringen, wenn du noch nicht damit vertraut bist, Integrationen wie UIX als Dienst einzurichten.

??? warning "UI Lovelace Minimalist lädt möglicherweise die Card-mod-Ressource"
    Wenn UI Lovelace Minimalist installiert ist, lädt es möglicherweise die Card-mod-Ressource. Wegen der Ladereihenfolge der Integrationen kann UI eXtension diesen Konflikt nicht erkennen. Deaktiviere in UI Lovelace Minimalist die Option `Include custom card resources it's depending on`, wenn du UI Lovelace Minimalist weiter mit UI eXtension verwenden möchtest. Lade benötigte benutzerdefinierte Karten stattdessen über HACS oder manuell.

    ![Option für benutzerdefinierte Karten in UI Minimalist](./assets/page-assets/faq/ui-minimalist-option.png)

## Ist UI eXtension ein direkter Ersatz für Card-mod?

Ja. UI eXtension ist ein direkter Ersatz für Card-mod bis Version 4.2.1. Alle Card-mod-Karten- und Theme-Konfigurationen werden unterstützt. Für neue Konfigurationen empfehlen wir `uix:` in Karten und `uix-<thing>(-yaml)` für Themes. Eine Umstellung ist jedoch nicht erforderlich.

## Ist UI eXtension nur Card-mod mit einer anderen Dokumentation?

Nein. UIX verwendet eine eigene Domain und den Konfigurationsschlüssel `uix`. Card-mod-Schlüssel werden weiterhin unterstützt, aber von `uix:` überschrieben.

??? info "Unterschiede zwischen UI eXtension und Card-mod"
    - Konfigurationsschlüssel für Karten: `uix:`.
    - Theme-Schlüssel: `uix-theme:`.
    - Theme-Schlüssel für einzelne Bereiche: `uix-<thing>(-yaml):`.
    - Das HTML-Element von UI eXtension ist `<uix-node>`; seine Eigenschaften beziehen sich auf `uix`.
    - Mit `{# uix.debug #}` kannst du Templates debuggen.
    - Alle Meldungen in der Debug-Konsole beginnen mit `UIX`.

## Welche Unterschiede gibt es zwischen Card-mod und UI eXtension?

Die folgende Tabelle fasst die Funktionen zusammen.

<!-- markdownlint-disable MD033 -->
| Funktion | Card-mod | UIX |
| --- | :---: | :---: |
| `...-yaml`-Theme-Variablen korrekt laden | ❌<br>Seit 2026.8.0 | Ja |
| Theme-Variable `...-more-info(-yaml)` korrekt verarbeiten | ❌<br>Seit 2026.3.0 | Ja |
| Adaptive Dialoge für die Theme-Variable `...-dialog(-yaml)` korrekt anpassen | ❌<br>Seit 2026.3.0 | Ja |
| [DOM-Prüfwerkzeuge](./concepts/dom.md#dom-inspection-helpers) | Nein | Ja |
| [Auswahl von Host- und Elementpfaden](./concepts/dom.md#hostelement-path-selection) | Nein | Ja |
| [Express-Suchselektor](./concepts/dom.md#express-search-selector) | Nein | Ja |
| [Forge](./forge/index.md) (benutzerdefiniertes Lovelace-Element) | Nein | Ja |
| [Forge – Foundries](./forge/foundries.md) (wiederverwendbare Forges) | Nein | Ja |
| [Forge – Makros](./using/templates.md#macros) (wiederverwendbare Jinja-Templates) | Nein | Ja |
| [Forge – Sparks](./forge/sparks/index.md) (eigenständige Verhaltensweisen für Forge-Elemente) | Nein | Ja |
| [Broker](./broker/index.md) (deklarative Frontend-Ereignisinteraktionen) | Nein | Ja |
| [Drosselung von Frontend-Zuständen](./extras/frontend-states-throttling.md) (optional) | Nein | Ja |
| [Verzögerung beim Styling von Dialogen](./extras/dialog-styling-delay.md) (optional) | Nein | Ja |
| [Hintergründe für Dashboard-Ansichten](./using/view-backgrounds.md) | Nein | Ja |
| [Abschnittshintergründe](./using/section-backgrounds.md) | Nein | Ja |
| [Ansichtshintergründe](./using/view-backgrounds.md) | Nein | Ja |
| [Icon-Styling – Entity-Überschreibung](./using/icons.md#specifying-for-an-entity-override) | Nein | Ja |
| [Entity-Bilder stylen](./using/images.md) | Nein | Ja |
| [Benutzerdefinierte Panels stylen](./using/custom-panels.md), auch in iFrames geladene | Nein | Ja |
| [App- und Ingress-Panels stylen](./using/apps.md), auch in iFrames geladene | Nein | Ja |
| Popup zum Neuladen und Cache-Leeren | Nein | Ja |
| Ausführliche Dokumentation mit visuellen Beispielen | Teilweise | Ja |
| Mod-Card | Ja | Ja |
| CSS-Styling in Themes | Ja | Ja |
| Dienst/Aktion zum Neuladen und Cache-Leeren | Ja | Ja |
| Stellt Variablen bereit, zum Beispiel den aktuellen Benutzer | Ja | Ja |
| CSS-Styling | Ja | Ja |
| Ressourcen-URL | Ja | Nicht zutreffend |

## Gibt es Probleme mit Ressourcen-URLs in UI eXtension?

Nein. Als Integration verwaltet UI eXtension die Ressourcen-URLs selbst. Du musst nichts tun, damit UI eXtension optimal läuft. UI eXtension fügt die Frontend-Ressource `uix.js` dynamisch als zusätzliches Modul hinzu und registriert außerdem eine Dashboard-Ressource, wenn du CAST verwendest. Bei jedem Laden der Integration wird die jeweilige Version automatisch an diese Ressourcen angehängt.

## Muss ich nach einem Update den Cache von Browsern und Companion-Apps manuell leeren?

UI eXtension zeigt eine Meldung an, wenn zum Leeren des Caches ein Neuladen nötig ist. Du kannst über die Schaltfläche `Reload Now` direkt neu laden; andernfalls wird nach 60 Sekunden automatisch neu geladen.

!!! note
    Der Code für das automatische Neuladen ist ab Version 8.1.0 enthalten, wird aber erst nach dem nächsten Update verfügbar sein. Nach der Installation von 8.1.0 läuft auf dem Gerät zunächst noch der Code aus Version 8.0.1, der diese Funktion nicht enthält.

## Wie deinstalliere ich UI eXtension?

Die Deinstallation erfolgt in zwei Schritten. Entferne zuerst den Diensteintrag unter **Geräte & Dienste**. Deinstalliere anschließend die Integration über HACS oder entferne bei einer manuellen Installation den Ordner `uix` aus dem Verzeichnis `custom_components`.
