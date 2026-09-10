---
title: UIX Broker
description: Deklarative Frontend-Ereignisinteraktionen für Home Assistant mit UIX Broker erstellen.
---
# UIX Broker

UIX Broker wandelt Browser-Ereignisse, Tastaturkürzel und Home-Assistant-Event-Bus-Ereignisse in deklarative Interaktionen um. Eine Interaktion wählt ein Browser-Element aus, prüft optionale Regeln und führt anschließend Direktiven in der konfigurierten Reihenfolge aus.

```text
Realm → Listen → Interaktionsanker → Regeln (optionale Anker) → Direktiven (optionale Anker)
```

Nutze UIX Broker, wenn sich ein Verhalten der Oberfläche konfigurieren lässt, statt dafür eine eigene Card, ein Script oder einen Patch zu schreiben. UIX Broker kann auf Klicks reagieren, ein Ereignis vor dem erneuten Auslösen anpassen, ein Element fokussieren, eine Objekteigenschaft aktualisieren, eine sichere Elementmethode aufrufen, eine interaktive Schaltfläche hinzufügen und JavaScript-Aktionen mit verfügbaren Interaktionsvariablen ausführen.

```yaml
uix_broker:
  - realm: browser
    listen: click
    anchor: target
    rules:
      - ".action-button"
    directives:
      - type: block
      - type: event
        name: another-action
        data:
          source: action-button
```

## UIX-Broker-Anleitungen

- [Broker](./broker.md) – Interaktionsstruktur, Konfigurationsquellen, Lebenszyklus und Debugging.
- [Realms](./realms.md) – Browser-Ereignisse, Tastaturkürzel und Home-Assistant-Event-Bus-Ereignisse.
- [Interaktionsanker](./interaction-anchors.md) – Auswahl von Elementen über zusammengesetzten Ereignispfad und `select_tree`.
- [Regeln](./rules.md) – Abgleich von Host-Element, erfassten Daten und Browser-Identität.
- [Direktiven](./directives.md) – `block`, `property`, `event`, `call`, `button` und Home-Assistant-Aktionen.
- [Beispiele](./examples.md) – Beispiele. Siehe auch [UIX Guides](https://uix-guides.lf.technology), wo weitere ausführliche Beispiele veröffentlicht werden können.

!!! note
    Für den Abgleich der Browser-Identität ist [Browser Mod](https://github.com/thomasloven/hass-browser_mod) erforderlich.

## Zukünftige Funktionen

UIX Broker befindet sich in aktiver Entwicklung. Alle bisherigen Funktionen und Beispiele sind aus Ideen entstanden, die Nutzer im Community-Forum geteilt haben. Wenn du eine Idee hast, wie UIX Broker erweitert werden kann, starte bitte eine [GitHub-Diskussion](https://github.com/Lint-Free-Technology/uix/discussions). Funktionen mit 10 Upvotes können als Feature Request in den UIX-GitHub-Issue-Tracker übernommen werden.

Geplante zukünftige UIX-Broker-Funktionen sind:

- **JavaScript-Regel**: führt JavaScript aus und stellt den aktuellen Interaktionszustand als Variablen bereit. Gibt ein Objekt im Format `{result: <truthy>, [optional] namedObject: <object data>}` zurück; das optionale `namedObject` ist anschließend für weitere Regeln und alle Direktiven verfügbar.
- **Erweiterte JavaScript-Aktionsdirektive**: unterstützt Rückgaben aus der aktuellen JavaScript-Aktionsdirektive. Rückgabeformat: `{continue: <truthy>, [optional] namedObject: <object data>}`. Wenn `continue` false ist, werden keine weiteren Direktiven ausgeführt. Das optionale `namedObject` ist anschließend für die übrigen Direktivenoperationen verfügbar.
- **Jinja2-Template-Regel**: rendert ein einmaliges Jinja2-Template, das ein truthy-Ergebnis zurückgibt und optional Objektdaten liefern kann.
