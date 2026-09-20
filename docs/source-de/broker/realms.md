---
title: Realms
description: Wähle aus, wo eine UIX-Broker-Interaktion auf Ereignisse hört.
---
# Realms

Der `realm` einer Interaktion bestimmt, wo Broker lauscht und wie der Wert von `listen` interpretiert wird.

| Realm | `listen` | Ereignisquelle | Anker-Unterstützung |
| --- | --- | --- | --- |
| `browser` | DOM-Ereignisname, z. B. `click` oder `show-dialog` | Browser-Ereignisse auf `window` während der Capture-Phase | Event-Path-Ausdrücke aus dem zusammengesetzten Pfad und `select_tree`-Pfade |
| `shortcut` | [Tinykeys](https://jamiebuilds.github.io/tinykeys/)-Tastenkombination, z. B. `"$mod+Shift+K"` | Browser-Tastaturereignis | Event-Path-Ausdrücke aus dem zusammengesetzten Pfad und `select_tree`-Pfade |
| `server` | Home-Assistant-Event-Bus-[Ereignis](https://www.home-assistant.io/docs/configuration/events/), z. B. `state_changed`, `component_loaded` oder `call_service` | Aktive Frontend-Verbindung | Nur `select_tree`-Pfade |

Alle Realms unterstützen [Regeln](rules.md) und [Direktiven](directives.md). Das ausgewählte Element des [Interaktionsankers](interaction-anchors.md) befindet sich immer im aktuellen Browser. Daher kann ein aus dem `server`-Realm erfasstes Ereignis trotzdem ein Browser-Element aktualisieren oder ein Ereignis an dieses Element auslösen.

## Listener-Registrierungen

Broker registriert einen Browser-Listener für jeden eindeutigen Ereignisnamen im `browser`-Realm und ein Home-Assistant-Event-Bus-Abonnement für jeden eindeutigen Ereignisnamen im `server`-Realm. Interaktionen mit demselben aktivierten `listen`-Wert teilen sich diese Registrierung. Broker wertet ihre Anker und Regeln aus, nachdem das Ereignis angekommen ist.

Zum Beispiel verwenden zwei `browser`-Interaktionen, die beide auf `uix-applied` lauschen, einen gemeinsamen `window`-Listener. Zwei `server`-Interaktionen, die beide auf `state_changed` lauschen, verwenden ein gemeinsames Event-Bus-Abonnement. Teile Interaktionen nach Ziel, Regeln oder Direktiven auf, wenn die Konfiguration dadurch klarer wird; eine Gruppierung ist nicht nötig, um Listener-Registrierungen zu reduzieren.

## Browser

`browser` lauscht während der Capture-Phase auf `window`. Nutze diesen Realm für DOM-Ereignisse wie `click`, `change`, `show-dialog` und benutzerdefinierte Browser-Ereignisse von Home Assistant. `listen` kann ein einzelner Ereignisname oder eine Liste sein, wenn dieselbe Interaktion auf mehrere Ereignisse reagieren soll.

```yaml
- realm: browser
  listen: show-dialog
  anchor: '&home-assistant $ hui-dialog-create-card'
  debug: true
  rules:
    - '@captured.dialogTag': hui-dialog-create-card
  directives:
    - type: property
      set: _currTab
      value: card
```

Beispiel: Eine Interaktion nach dem Start von Broker und jedes Mal ausführen, wenn ein Panel-Update `uix-update` auslöst:

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

Das `detail`-Objekt des Browser-Ereignisses ist die Wurzel der erfassten Daten. Siehe [Regeln für erfasste Daten](./rules.md#regeln-für-erfasste-daten) und [Event-Direktive](./directives.md#event), um zu erfahren, wie erfasste Daten abgeglichen und wiederverwendet werden.

### UIX-Styling-Lifecycle-Events

UIX Styling löst die folgenden bubbling und composed Browser-Ereignisse von seinem `<uix-node>` aus:

- `uix-applied` – nachdem UIX an ein Element angehängt oder erneut angewendet wurde. Das Ereignis kann erneut ausgelöst werden, wenn der Host aktualisiert oder die UIX-Konfiguration erneut angewendet wird. Verbraucher sollten ihre Direktiven daher idempotent gestalten.
- `uix-styles-update` – wenn dieser UIX-Knoten seinen gerenderten CSS-Text aktualisiert, einschließlich templategesteuerter Updates. Der neueste Text steht als `detail.uix_node._rendered_styles` bereit, aber Lit hat sein `<style>`-Element zu diesem Zeitpunkt noch nicht committed. Um berechnete Styles zu lesen, warte zuerst in einer JavaScript-Direktive auf `detail.uix_node.updateComplete`.
- `uix-theme-update` – nachdem dieser UIX-Knoten ein Theme-Update erneut verarbeitet hat. Dieses Ereignis wird auch ausgelöst, wenn das daraus resultierende UIX-CSS unverändert bleibt.

Alle drei Ereignisse liefern den auslösenden `<uix-node>` als `detail.uix_node`. Nutze den Event-Path-Anker `"< target"`, um dessen Elternelement auszuwählen. Das ist das Element, auf das UIX angewendet wurde, unabhängig davon, ob sich der Knoten im Light DOM oder in einem shadow root befindet.

Beispiel: Den `themeMode` einer Karte setzen, wenn UIX auf die umgebende Karten-Card angewendet wird und wenn sich ihr Theme aktualisiert:

```yaml
- realm: browser
  listen:
    - uix-applied
    - uix-theme-update
  anchor: "< target"
  rules:
    - hui-map-card
  directives:
    - type: property
      anchor: "$ ha-map"
      set: themeMode
      value: dark
```

## Shortcut

`shortcut` verwendet [Tinykeys](https://jamiebuilds.github.io/tinykeys/), um eine Browser-Tastenkombination auf `window` zu registrieren. Der Wert von `listen` nutzt die Tinykeys-Syntax. `$mod` bedeutet auf macOS `Meta` und unter Windows sowie Linux `Control`.

```yaml
- realm: shortcut
  listen: "$mod+Shift+Y"
  anchor: target
  directives:
    - type: call
      method: focus
```

Tastenkombinationen können eine Taste, einen Code, Modifikatoren und Sequenzen verwenden. Home Assistant nutzt Tinykeys ebenfalls für eigene Tastenkürzel. Verwende Tastenkombinationen, die nicht mit Home Assistant, dem Browser oder dem Betriebssystem kollidieren. Du kannst Home-Assistant-Tastaturkürzel für den Browser deaktivieren, um diese Tastenkombinationen für UIX Broker verfügbar zu machen; UIX Broker registriert Tastenkombinationen weiterhin, wenn Home-Assistant-Tastaturkürzel deaktiviert sind.

Das auslösende `KeyboardEvent` steht JavaScript-Aktionen als `event` zur Verfügung. Sein zusammengesetzter Pfad kann außerdem von [Interaktionsankern](./interaction-anchors.md) verwendet werden.

## Server

`server` abonniert den Home-Assistant-Event-Bus über die aktive Frontend-Verbindung. Es gibt kein Browser-Ereignisziel, daher muss der Interaktionsanker `select_tree`-Pfade verwenden.

```yaml
- realm: server
  listen: state_changed
  anchor: "&home-assistant $$ dynamic-custom-card"
  rules:
    - type: captured
      path: data.entity_id
      match: light.kitchen
  directives:
    - type: property
      set: customCardProperty
      value: "@captured.data.entity_id"
```

Bei Server-Interaktionen enthalten erfasste Daten einen Schlüssel `data` mit der Home-Assistant-Ereignisnutzlast, zum Beispiel `data.entity_id` oder `data.new_state.state`. In Regeln und Direktiven können erfasste Daten mit `"@captured.data..."` referenziert werden; die Anführungszeichen sind in YAML wegen `@` erforderlich.

## Blockieren

Die Direktive `block` ist in den Realms `browser` und `shortcut` verfügbar. Ihr Anker und Host-Element-Regeln werden synchron aufgelöst. Wenn ein `select_tree`-Anker nicht sofort gefunden wird, wird die komplette Interaktion übersprungen. Dadurch bleiben Timing von Browser-Propagation und Standardaktion erhalten. Server-Ereignisse können nicht blockiert werden.

!!! note
    Tinykeys ignoriert Tastendrücke in Eingabefeldern, Textareas, Selects und contenteditable-Bereichen. Eine `block`-Direktive im `shortcut`-Realm läuft in diesen Situationen daher nicht. Um eine Taste in diesen Situationen zu blockieren, nutze den `browser`-Realm mit `listen: keydown` zusammen mit Regeln für erfasste Daten und/oder synchronen Host-Element-Regeln.

    Wenn ein Shortcut-Binding ausgeführt wird, verhindert `block` die native Standardaktion und stoppt spätere Listener auf `window`. Es kann keinen Home-Assistant-Shortcut-Handler rückgängig machen, der bereits gelaufen ist.

## Templates

UIX Broker stellt bewusst keinen Realm bereit, der direkt Jinja2-Templates abonniert. Die [`template`-Direktive](./directives.md#template) kann ein Template einmalig rendern, während eine Interaktion läuft, lauscht aber nicht auf spätere Änderungen. Für reaktives Verhalten nutze ein Script, eine Automation oder eine Template-Entität mit Trigger; löse anschließend ein benutzerdefiniertes Ereignis auf dem Home-Assistant-Event-Bus aus und lausche im `server`-Realm darauf.

!!! tip
    Du kannst die Integration [`custom_event`](https://github.com/reubn/hass_custom_event) verwenden, um benutzerdefinierte Ereignisse auf dem Home-Assistant-Event-Bus auszulösen und anschließend im `server`-Realm darauf zu lauschen.
