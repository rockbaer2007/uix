---
title: UIX Broker
description: Deklarative Frontend-Ereignisinteraktionen für Home Assistant mit UIX Broker erstellen.
---
# UIX Broker

UIX Broker wandelt Browser-Ereignisse, Tastaturkürzel und Home-Assistant-Event-Bus-Ereignisse in deklarative Interaktionen um. Eine Interaktion wählt ein Browser-Element aus, prüft optionale Regeln und führt anschließend Direktiven in der konfigurierten Reihenfolge aus. Die Direktive `block` ist eine Ausnahme: Sie blockiert das auslösende Ereignis synchron, bevor die übrigen Direktiven laufen.

```text
Realm → Listen → Interaktionsanker → Regeln (optionale Anker) → Direktiven (optionale Anker)
```

Nutze UIX Broker, wenn sich ein Verhalten der Oberfläche konfigurieren lässt, statt dafür eine eigene Card, ein Script oder einen Patch zu schreiben. UIX Broker kann auf Klicks reagieren, ein Ereignis vor dem erneuten Auslösen anpassen, ein Element fokussieren, eine Objekteigenschaft aktualisieren und eine sichere Elementmethode aufrufen. Außerdem kann Broker Schaltflächen, Badges, Text, Tile-Icons, Tooltips und Entsperr-Abfragen hinzufügen, Home-Assistant-Aktionen binden oder ausführen, Templates rendern, JavaScript auswerten und Abläufe pausieren.

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
- [Regeln](./rules.md) – Abgleich von Host-Element, erfassten Daten, Browser-Identität, Benutzer, Administratorstatus, URL-Fragment, Suchparametern und Panel.
- [Direktiven](./directives.md) – `block`, `property`, `event`, `call`, `button`, `badge`, `text-content`, `tile-icon`, `tooltip`, `lock`, `action-handler`, `action`, `template`, `javascript` und `wait`.
- [Beispiele](./examples.md) – Beispiele. Siehe auch [UIX Guides](https://uix-guides.lf.technology), wo weitere ausführliche Beispiele veröffentlicht werden können.

!!! note
    Für den Abgleich der Browser-Identität ist [Browser Mod](https://github.com/thomasloven/hass-browser_mod) erforderlich.
