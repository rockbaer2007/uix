---
title: Broker
description: UIX-Broker-Interaktionen konfigurieren und ihre Konfigurationsquellen verwalten.
---
# UIX Broker

Eine Interaktion besitzt einen `realm`, einen `listen`-Wert, einen Interaktions-`anchor`, optionale `rules` und eine geordnete Liste von `directives`.

```yaml
uix_broker:
  - realm: shortcut
    debug: true
    listen: "$mod+Shift+Y"
    anchor: '&home-assistant'
    directives:
      - type: action
        action: fire-dom-event
        uix:
          action: toast
          data:
            message: Shortcut pressed
```

Siehe [Realms](./realms.md), [Interaktionsanker](./interaction-anchors.md), [Regeln](./rules.md) und [Direktiven](./directives.md) für die einzelnen Teile einer Interaktion.

## Interaktionsoptionen

| Schlüssel | Beschreibung |
| --- | --- |
| `realm` | Wo UIX lauscht: `browser`, `shortcut` oder `server`. |
| `listen` | Der DOM-Ereignisname, die [Tinykeys](https://jamiebuilds.github.io/tinykeys/)-Tastenkombination oder der Name eines Home-Assistant-Event-Bus-Ereignisses für den gewählten Realm. Im `browser`-Realm kann dies auch eine Liste von DOM-Ereignisnamen sein. |
| `anchor` | Das Element, das geprüft und als Standardziel für Regeln und Direktiven verwendet wird. |
| `rules` | Optionale Bedingungen, die alle passen müssen, bevor Direktiven ausgeführt werden. |
| `directives` | Geordnete Operationen, die ausgeführt werden, wenn die Interaktion passt. |
| `enabled` | Standardmäßig `true`. Auf `false` setzen, um eine Interaktion in der Konfiguration zu behalten, ohne sie zu registrieren. |
| `reentrant` | Standardmäßig `true`. Auf `false` setzen, um passende Ereignisse für dieselbe Interaktion zu ignorieren, solange sie aufgelöst oder ausgeführt wird. |
| `debug` | Auf `true` setzen, um den Lebenszyklus der Interaktion in der Browser-Entwicklerkonsole zu protokollieren. |

Jede Interaktion ist unabhängig. Alle ihre Regeln müssen passen, bevor Direktiven ausgeführt werden; Direktiven laufen einzeln in der Reihenfolge der Konfiguration.

Nutze eine `listen`-Liste im Browser-Realm, wenn dieselbe Interaktion für mehr als ein Browser-Ereignis laufen soll:

```yaml
- realm: browser
  listen:
    - uix-broker-ready
    - uix-update
  anchor: '&home-assistant'
  directives:
    - type: call
      method: requestUpdate
```

Listen werden nur im `browser`-Realm unterstützt. `shortcut`- und `server`-Interaktionen lauschen jeweils auf eine einzelne Tastenkombination oder einen einzelnen Ereignisnamen.

`reentrant: false` ist nützlich, wenn eine Interaktion dasselbe Ereignis auslöst, das sie gestartet hat. Die Interaktion gilt als aktiv, während Anker aufgelöst werden, Direktiven laufen und Wartezeiten von Direktiven aktiv sind.

## Broker-ready-Ereignis

Nachdem UIX Broker seine Konfiguration angewendet hat, löst er auf `window` ein Browser-Ereignis `uix-broker-ready` aus. Das Ereignis wird ausgelöst, nachdem Broker seine Listener im Browser-Realm registriert hat. Eine Interaktion kann daher auf dieses Ereignis lauschen, um eine anfängliche UI-Anpassung vorzunehmen. Es wird außerdem nach jedem erneuten Laden der Broker-Konfiguration ausgelöst.

Ein Beispiel für dieses Ereignis findest du unter [Tools-Schaltfläche zum Sidebar-Titel hinzufügen](./examples.md#tools-schaltfläche-zum-sidebar-titel-hinzufügen).

## Konfigurationsquellen

Konfiguriere Interaktionen auf eine oder mehrere der folgenden Arten:

1. Im UIX-Optionsflow – **Einstellungen → Geräte & Dienste → UIX → Konfigurieren (Zahnrad) → Broker konfigurieren**.
1. In einer oder mehreren registrierten YAML-Dateien – **Einstellungen → Geräte & Dienste → UIX → Konfigurieren (Zahnrad) → Broker-Dateien verwalten**.

Jede YAML-Datei ist ein Mapping mit einer `uix_broker`-Liste auf oberster Ebene:

```yaml
uix_broker:
  - realm: browser
    listen: click
    anchor: target
    rules:
      - home-assistant
    directives:
      - type: block
```

Nutze **Broker-Dateien verwalten**, um Dateien zu registrieren, zu deregistrieren oder neu zu laden. Dateipfade dürfen absolut oder relativ zum Home-Assistant-Konfigurationsverzeichnis sein. Registrierte Dateien werden in Registrierungsreihenfolge gelesen, anschließend werden über die UI konfigurierte Interaktionen angehängt. Alle Interaktionen werden als eine Liste an verbundene Browser ausgeliefert.

YAML-Dateikonfigurationen verwenden dieselbe Home-Assistant-YAML-Auflösung wie Foundries, einschließlich `!include` und `!secret`. Die Aktion **UIX Broker** unter **Werkzeuge → YAML** lädt alle registrierten Broker-Dateien neu und meldet Dateifehler. Bei Dashboards im YAML-Modus lädt auch die eingebaute Aktion **Aktualisieren** des Dashboards registrierte Broker-Dateien neu.

## Synchrone und asynchrone Ausführungspfade von Interaktionen

Regeln für erfasste Daten und Browser-Identität laufen synchron vor der Auflösung des Interaktionsankers. Event-Path-Interaktionsanker werden ebenfalls synchron aufgelöst. Dadurch kann eine Interaktion im Browser-Realm eine `block`-Direktive anhand erfasster Daten, Browser-Identität und bereits im zusammengesetzten Ereignispfad vorhandener Elemente anwenden.

Da die [`block`-Direktive](directives.md) synchron laufen muss, benötigen Interaktionen mit `block`, dass ihre [Interaktionsanker](interaction-anchors.md) und Anker von [Host-Element-Regeln](rules.md#host-element-regeln) sofort verfügbar sind. UIX Broker führt eine synchrone Suche aus; wenn einer davon nicht verfügbar ist, wird die Interaktion übersprungen.

Nachdem eine blockierende Interaktion aufgelöst und `block` angewendet wurde, verwenden Anker späterer `property`-, `event`-, `call`- und `button`-Direktiven weiterhin das normale asynchrone Wiederholungsverhalten.

Bei Interaktionen ohne `block` werden fehlende [Interaktionsanker](interaction-anchors.md) und Anker von [Host-Element-Regeln](rules.md#host-element-regeln) alle 50 ms bis zu zwei Sekunden lang erneut gesucht. Dadurch kann eine Interaktion, die auf ein Browser-Ereignis wie `show-dialog` lauscht, warten, bis der Dialog gemountet wurde, bevor der Dialog oder eines seiner Elemente als Interaktionsanker ausgewählt wird.

Weitere Informationen findest du unter [Realms](realms.md), [Interaktionsanker](interaction-anchors.md) und [Regeln](rules.md).

## Debugging

Setze `debug: true` für eine Interaktion, um Listener-Aktivität, Ankerauflösung, jedes Regelergebnis sowie jede Direktive vor und nach ihrer Ausführung zu protokollieren. Der Eintrag nach der Ausführung einer `template`- oder `javascript`-Direktive enthält außerdem ihr gespeichertes Ergebnis. Debug-Logmeldungen sind mit Realm und `listen`-Wert der Interaktion gekennzeichnet.

```yaml
- realm: browser
  listen: click
  anchor: target
  debug: true
  rules:
    - ".action-button"
  directives:
    - type: event
      name: another-event
```

## Reaktivität von Interaktionen

UIX-Broker-[`template`- und `javascript`-Direktiven](directives.md) laufen nur, wenn ihre Interaktion läuft; keine der beiden abonniert Zustandsänderungen. Wenn eine Interaktion auf Entity-State-Updates reagieren soll, erstelle eine Hilfsinteraktion, die auf `state_changed` lauscht, mit einer [Regel](./rules.md) für die gewünschte Entity. Nutze dann eine `event`-Direktive, um ein benutzerdefiniertes Browser-Ereignis auszulösen, und füge dieses Ereignis zur `listen`-Liste der eigentlichen Interaktion hinzu.

Server-Realm-zu-Browser-Realm-Interaktion:

```yaml
  - realm: server
    listen: state_changed
    anchor: "&home-assistant"
    directives:
      - type: event
        name: uix-update-my-interaction
        rules:
          - type: captured
            path: data.entity_id
            match:
              or:
                - switch.bed_light
                - light.bed_light
```

Browser-Realm-Interaktion:

```yaml
  - realm: browser
    listen:
      - uix-broker-ready
      - uix-update-my-interaction
    anchor: "&home-assistant $ home-assistant-main $ ha-sidebar"
    #... rules and directives
```

Ein vollständiges Beispiel findest du unter [Light-Schaltfläche zum Home-Dashboard-Menüeintrag in der Sidebar](./examples.md#light-schaltfläche-zum-home-dashboard-menüeintrag-in-der-sidebar).
