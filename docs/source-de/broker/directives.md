---
title: Direktiven
description: Deklarative UIX-Broker-Operationen auf ein ausgewähltes Element anwenden.
---
# Direktiven

Direktiven laufen nacheinander, nachdem alle Interaktionsregeln gepasst haben. Jede Direktive führt eine konfigurierte Operation aus und verwendet standardmäßig den Interaktionsanker oder, sofern unterstützt, einen ausdrücklich ausgewählten Direktivenanker. Mit Ausnahme von `block` kann eine Direktive außerdem eigene `rules` besitzen. Die Direktive läuft nur, wenn alle diese Regeln passen; andernfalls überspringt Broker sie und fährt mit der nächsten Direktive fort.

- [Block](#block) – Standardaktion und Propagation des auslösenden Browser-Ereignisses verhindern.
- [Property](#property) – eine JavaScript-Objekteigenschaft setzen oder löschen.
- [Event](#event) – ein `CustomEvent` auslösen.
- [Call](#call) – eine Elementmethode aufrufen.
- [Button](#button) – eine interaktive Home-Assistant-Schaltfläche einfügen.
- [Textinhalt](#textinhalt) – gestalteten Text neben einem Element einfügen.
- [Tile icon](#tile-icon) – ein interaktives Home-Assistant-Tile-Icon einfügen.
- [Tooltip](#tooltip) – einen gestalteten Tooltip an ein Element anhängen.
- [Lock](#lock) – eine Entsperr-Abfrage verlangen, bevor ein Element verwendet werden kann.
- [Action](#action) – eine Home-Assistant-, Frontend- oder UIX-Aktion ausführen.
- [Template](#template) – ein Jinja2-Template einmalig rendern und das Ergebnis speichern.
- [JavaScript](#javascript) – JavaScript synchron auswerten und den Rückgabewert speichern.
- [Wait](#wait) – die nächste Direktive verzögern.

## Direktivenregeln

Füge `rules` zu jeder Direktive außer `block` hinzu, um nur diese Direktive zu bedingen. Die Syntax entspricht den [Interaktionsregeln](./rules.md). Bei `property`, `event`, `call`, `action-handler`, `button`, `badge`, `text-content`, `tile-icon`, `tooltip` und `lock` prüfen Host-Element-Regeln standardmäßig den aufgelösten Direktivenanker. Bei `action` und `wait` prüfen sie den Interaktionsanker. Der eigene `anchor` einer Regel bleibt relativ zu diesem Standardanker oder kann wie üblich absolut sein.

```yaml
directives:
  - type: property
    set: config.mode
    value: advanced
  - type: call
    method: openAdvancedEditor
    rules:
      - type: captured
        path: allow_advanced
        match: true
```

`panel`-Regeln rufen den aktuellen Panel-Zustand ab, wenn die Direktive erreicht wird. Dadurch kann eine frühere Direktive unabhängig vom aktuellen Panel laufen, während eine spätere Direktive nur auf einem passenden Panel ausgeführt wird.

`block` akzeptiert keine Direktivenregeln. Lege seine Bedingung in die `rules` der Interaktion, damit das Ereignis nur dann synchron blockiert wird, wenn die komplette Interaktion passt.

## Block

`block` ruft `preventDefault()` und `stopImmediatePropagation()` auf dem auslösenden Browser-Ereignis auf.

```yaml
- type: block
```

Diese Direktive ist nur in den Realms `browser` und `shortcut` verfügbar. Der Interaktionsanker und Anker von Host-Element-Regeln müssen synchron auflösbar sein. Wenn ein erforderlicher `select_tree`-Anker noch nicht vorhanden ist, überspringt UIX Broker die komplette Interaktion. Eine `block`-Direktive wird angewendet, bevor die übrigen Direktiven verarbeitet werden, auch wenn sie später in der Liste steht.

## Direktivenanker

Die Direktiven `property`, `event`, `call`, `action-handler`, `button`, `badge`, `text-content`, `tile-icon`, `tooltip` und `lock` verwenden standardmäßig den Interaktionsanker. Jede kann diesen Standard mit einer eigenen `anchor`-Konfiguration überschreiben. Eine einfache Zeichenfolge ist relativ zum Interaktionsanker, eine Zeichenfolge mit führendem `&` ist ein kompakter absoluter `select_tree`-Pfad ab dem Dokument-Root, und `{ select_tree: ... }` ist die entsprechende lange absolute Form.

```yaml
directives:
  - type: property
    anchor: "$ ha-dialog"
    set: withoutHeader
    value: true
  - type: event
    anchor: "&home-assistant $$ ha-automation-sidebar"
    name: broker-sidebar-event
  - type: call
    anchor:
      select_tree: "home-assistant $ ha-more-info-dialog"
    method: closeDialog
```

Nutze den Konsolenhelfer `uix_broker_path($0)` in der Browser-Konsole, um einen relativen Direktivenankerpfad zu finden.

Siehe [Interaktionsanker](./interaction-anchors.md#anker-in-regeln-und-direktiven) für die Auswahlformate.

Weitere Informationen zu den verfügbaren Konsolenhelfern findest du unter [Pfade in der Browser-Konsole finden](./interaction-anchors.md#pfade-in-der-browser-konsole-finden).

## Property

Die `property`-Direktive ändert das JavaScript-Objekt des ausgewählten Ankers. `set` nimmt einen durch Punkte getrennten Eigenschaftspfad, erstellt fehlende Zwischenebenen als einfache Objekte und weist den Wert an der letzten Eigenschaft zu. `clear` nimmt dieselbe Art von Pfad und löscht nur die letzte Eigenschaft; übergeordnete Objekte werden nicht entfernt.

```yaml
- type: property
  set: config.heading
  value: New title
- type: property
  clear: config.icon
```

Werte können auf erfasste Daten oder ein vorheriges `template`- oder `javascript`-Ergebnis verweisen. `@captured` löst auf das vollständige Objekt erfasster Daten auf, während `@captured.path` auf den Wert an diesem durch Punkte getrennten Pfad auflöst. Array-Indizes können Punktnotation (`items.0`) oder Klammern (`items[0]`) verwenden. Für Objekteigenschaften mit Satzzeichen, zum Beispiel `settings['icon-color']`, nutzt du einen zitierten Klammer-Schlüssel. Die Referenz wird ersetzt, bevor die Eigenschaft gesetzt wird, und muss in YAML in Anführungszeichen stehen, weil sie mit `@` beginnt.

```yaml
- type: property
  set: config.entity
  value: "@captured.entity_id"
```

`template`- und `javascript`-Direktiven speichern ihren Wert unter ihrer `id`. Eine spätere Direktive kann `@id` oder eine Eigenschaft wie `@id.path` verwenden. Der Wert behält seinen ursprünglichen Typ, einschließlich Objekten und Arrays. Referenzen belegen einen vollständigen YAML-Wert; Broker interpoliert sie nicht in längere Zeichenfolgen.

## Event

`event` löst ein `CustomEvent` aus. `target` ist standardmäßig `anchor`, also der ausgewählte Direktivenanker oder der Interaktionsanker, wenn kein Direktivenanker gesetzt ist. Setze `target: window` oder `target: document`, um global auszulösen; diese Ziele verwenden keinen ereignisspezifischen Direktivenanker und lösen keinen auf. `bubbles` und `composed` sind standardmäßig `false`, passend zur DOM-API.

```yaml
- type: event
  name: broker-demo-event
  bubbles: true
  composed: true
  data:
    entity: light.bed_light
```

```yaml
- type: event
  target: window
  name: broker-window-event
  data:
    source: uixBroker
- type: event
  target: document
  name: broker-document-event
```

Setze `capture_data: true`, um erfasste Ereignisdaten in ein verändertes Ereignis zu kopieren. Das `detail` des ausgehenden Ereignisses beginnt mit den erfassten Daten der auslösenden Interaktion und überschreibt diese anschließend flach mit Werten aus dem `data`-Objekt dieser Direktive. Die Option `capture_data` ist nur für die `event`-Direktive verfügbar.

```yaml
- type: event
  name: broker-forwarded-event
  capture_data: true
  data:
    source: uixBroker
```

Setze `capture_data: deep`, wenn verschachtelte einfache Objekte stattdessen zusammengeführt werden sollen. Direktiven-`data` gewinnt bei Konflikten; Arrays und nicht-einfache Objekte werden als vollständige Werte ersetzt. `capture_data: true` bleibt dadurch unverändert.

```yaml
- type: event
  name: broker-forwarded-event
  capture_data: deep
  data:
    params:
      source: uixBroker
```

## Call

`call` ruft eine Methode auf dem ausgewählten Anker auf. `method` akzeptiert einen sicheren, durch Punkte getrennten Methodenpfad und erhält das `this`-Binding des Methodenobjekts. `args` muss, wenn angegeben, ein Array sein und unterstützt Ersetzung durch erfasste Daten.

```yaml
- type: call
  method: focus
- type: call
  method: setSelectionRange
  args: [0, 5]
```

## Button

`button` fügt neben dem Direktivenanker ein Home-Assistant-`ha-button` ein. Die Direktive verwendet dieselbe Button-Konfiguration und Aktionsbehandlung wie der [Forge button spark](../forge/sparks/button.md). Standardmäßig wird die Schaltfläche nach dem Direktivenanker eingefügt.

Nutze `after` oder `before`, um ein anderes Referenzelement auszuwählen. Diese Pfade sind relativ zum aufgelösten Direktivenanker und unterstützen die übliche UIX-`select_tree`-Syntax. Die Schaltfläche wird weiterhin als Geschwisterelement des passenden Referenzelements eingefügt.

```yaml
- type: button
  label: Toggle
  entity: light.living_room
  tap_action:
    action: toggle
```

```yaml
- type: button
  anchor: "$ ha-dialog"
  before: "div.header"
  label: Toggle
  entity: light.living_room
  tap_action:
    action: toggle
```

Nutze `style` für ein flaches Mapping von CSS-Eigenschaftsnamen und Werten. Die Eigenschaften werden inline auf dem erzeugten `ha-button` gesetzt. Das ist nützlich für Button-Abmessungen und Abstände, die nicht aus der Dashboard-Konfiguration gestylt werden können.

```yaml
- type: button
  anchor: "$ div.menu div.title"
  icon: mdi:hammer
  color: red
  size: s
  tap_action:
    action: navigate
    navigation_path: /config/tools
  style:
    "--ha-button-box-shadow": rgba(0, 0, 0, 0.1) 0px 4px 12px
    "--ha-icon-button-size": 32px
```

Nutze `uix` für UIX Styling, einschließlich Styles innerhalb des shadow root der Schaltfläche. Der UIX-Typ ist `uix-broker-button`; die aufgelösten Button-Einstellungen stehen in UIX-Templates als `config` zur Verfügung, und Ergebnisse vorheriger `template`- oder `javascript`-Direktiven stehen als `directive` bereit.

!!! info
    `button`-UIX-Styling ist ab 8.3.0-beta.3 verfügbar

```yaml
- type: button
  entity: light.living_room
  label: Toggle
  uix:
    style: |
      :host {
        --uix-button-margin: {{ '6px' if is_state(config.entity, 'on') else '0px' }};
      }
```

| Schlüssel | Typ | Standard | Beschreibung |
| --- | --- | --- | --- |
| `after` | `string` | Direktivenanker | Relativer Selektor für das Referenzelement. Die Schaltfläche wird danach eingefügt. |
| `before` | `string` | — | Relativer Selektor für das Referenzelement. Die Schaltfläche wird davor eingefügt. |
| `entity` | `string` | — | Entity-ID, die von entitybasierten Aktionen verwendet wird. |
| `icon` | `string` | — | MDI-Icon, das im Label-Slot der Schaltfläche platziert wird. Es hat Vorrang vor `label`. |
| `color` | `string` | — | Icon-Farbe für eine reine Icon-Schaltfläche. |
| `label` | `string` | `""` | Button-Label. |
| `start_icon` / `end_icon` | `string` | — | MDI-Icon vor oder nach dem Label. |
| `variant` | `string` | Home-Assistant-Standard | `brand`, `neutral`, `danger`, `warning` oder `success`. Reine Icon-Schaltflächen verwenden standardmäßig `neutral`. |
| `appearance` | `string` | Home-Assistant-Standard | `accent`, `filled`, `outlined` oder `plain`. Reine Icon-Schaltflächen verwenden standardmäßig `plain`. |
| `size` | `string` | — | `s` (klein) oder `m` (mittel). |
| `style` | object | — | Flaches Mapping von CSS-Eigenschaftsnamen und String- oder Zahlenwerten; wird inline auf `ha-button` gesetzt. |
| `uix` | object | — | UIX-Konfiguration, die auf die erzeugte Schaltfläche als Typ `uix-broker-button` angewendet wird. |
| `tap_action` / `hold_action` / `double_tap_action` | action | — | Home-Assistant-Aktion, die von der Schaltfläche ausgeführt wird. |

!!! note
    - Setze höchstens eines von `after` und `before`.
    - Button-Klicks werden vom eigenen Action-Handler des Referenzelements isoliert.
    - Pointer-, Maus-, Touch- und Click-Ereignisse enden an der erzeugten Schaltfläche. Dadurch reagieren Ripple oder Action-Handler eines umschließenden Elements nicht, während die eigene Aktion und Ripple der Schaltfläche erhalten bleiben.
    - Es gilt dieselbe CSS-Variable `--uix-button-margin` wie beim Forge button spark. Der Standardabstand ist `-6px` für eine beschriftete Schaltfläche und `0px` für eine reine Icon-Schaltfläche.
    - Andere CSS-Variablen, die für den Forge button spark gelten, gelten ebenfalls.

## Badge

!!! info
    Die Direktive `badge` ist ab 8.3.0-beta.10 verfügbar.

`badge` fügt neben dem Direktivenanker ein `uix-badge` ein. Das Badge verwendet die von Home Assistant angepasste Web-Awesome-Basis und deren Styles; seine Varianten folgen daher dem aktiven Home-Assistant-Theme. UIX behält das Element im eigenen Namespace und registriert nicht die globale Web-Awesome-Komponente `wa-badge`.

Standardmäßig wird das Badge nach dem Direktivenanker eingefügt. Mit `after` oder `before` wird ein anderes Geschwisterelement über dieselbe UIX-`select_tree`-Syntax wie bei `button` gewählt. Ist die Referenz ein `ha-button` oder `ha-tile-icon`, zeigt UIX das Badge stattdessen automatisch auf diesem Element. Bei allen anderen Zielen positioniert `placement` das Badge auf dem Parent.

```yaml
- type: badge
  content: 3
  variant: danger
  appearance: filled
  pill: true
```

Verwende `style` für ein flaches Mapping aus CSS-Eigenschaftsnamen und Werten oder `uix` für UIX Styling. Der UIX-Typ ist `uix-broker-badge`; die aufgelösten Badge-Einstellungen stehen in UIX-Templates als `config`, Ergebnisse vorheriger `template`- oder `javascript`-Direktiven als `directive` bereit.

```yaml
- type: badge
  anchor: "$ div.title"
  before: ".label"
  content: Experimentell
  variant: warning
  appearance: outlined
  start_icon: mdi:flask-outline
  style:
    margin-inline-start: 8px
```

| Schlüssel | Typ | Standard | Beschreibung |
| --- | --- | --- | --- |
| `after` | Zeichenfolge | Direktivenanker | Relativer Selektor für die Referenz. Normalerweise wird das Badge danach eingefügt. Bei `ha-button` oder `ha-tile-icon` erscheint es auf diesem Element; mit `placement` wird es bei allen anderen Zielen auf dem Parent positioniert. |
| `before` | Zeichenfolge | — | Relativer Selektor für die Referenz. Normalerweise wird das Badge davor eingefügt. Bei `ha-button` oder `ha-tile-icon` erscheint es auf diesem Element; mit `placement` wird es bei allen anderen Zielen auf dem Parent positioniert. |
| `for` | `previous` | — | Unmittelbar nach einer Element erzeugenden Direktive verwenden, um ihr erzeugtes Element anzusprechen. Nicht mit `after` oder `before` kombinierbar. |
| `content` | Zeichenfolge oder Zahl | `""` | Im Badge angezeigter Text. |
| `variant` | Zeichenfolge | `brand` | `brand`, `neutral`, `success`, `warning` oder `danger`. |
| `appearance` | Zeichenfolge | `accent` | `accent`, `filled`, `outlined` oder `filled-outlined`. |
| `pill` | Boolean | `false` | Verwendet die vollständig abgerundete Pillenform. |
| `attention` | Zeichenfolge | `none` | `none`, `pulse` oder `bounce`. |
| `placement` | Zeichenfolge | — | Position auf `ha-button` oder `ha-tile-icon`; bei allen anderen Zielen auf deren Parent. Möglich sind `top`, `top-start`, `top-end`, `bottom`, `bottom-start`, `bottom-end`, `left`, `left-start`, `left-end`, `right`, `right-start` und `right-end`. |
| `start_icon` / `end_icon` | Zeichenfolge | — | MDI-Icon vor oder nach dem Inhalt. |
| `style` | Objekt | — | Flaches Mapping aus CSS-Eigenschaftsnamen und Zeichenfolgen- oder Zahlenwerten, inline auf `uix-badge` gesetzt. |
| `uix` | Objekt | — | UIX-Konfiguration für das erzeugte Badge mit Typ `uix-broker-badge`. |

### Automatische Platzierung

Die automatische Platzierung gilt nur, wenn der aufgelöste Direktivenanker oder die Referenz von `after` / `before` eines dieser Elemente ist. Ein von UIX erzeugter Button-Spark wird wie sein enthaltenes `ha-button` behandelt.

| Ziel | Platzierung | Umsetzung |
| --- | --- | --- |
| `ha-button` | Ecke oben rechts | UIX fügt das Badge innerhalb des Buttons ein und folgt dem Web-Awesome-Button-Badge-Muster. |
| `ha-tile-icon` | Ecke oben rechts | UIX verwendet den dokumentierten Standard-Slot des Tile-Icons, die Tile-Badge-Eckenversätze und die kompakte Tile-Badge-Größe von Home Assistant. |

Bei diesen Zielen bestimmen `after` und `before`, welches Element das Badge erhält; sie steuern nicht das Einfügen eines Geschwisterelements. Ohne `placement` verwenden andere Ziele das normale Geschwisterverhalten. Ein gesetztes `placement` positioniert das Badge relativ zum Parent: UIX belässt es außerhalb des Ziels und verändert das ausgewählte Ziel nicht.

Alle Platzierungen nutzen standardmäßig die kompakte Home-Assistant-Schriftgröße `--ha-font-size-xs` und `0.25em 0.5em` Innenabstand. Es gelten dieselben Werte wie für `wa-tooltip`. `ha-button` und `ha-tile-icon` verwenden `top-end` als Standard.

Die [CSS-Variablen des Forge-Badges](../forge/sparks/badge.md#css-variablen) können über `style` für ein einzelnes Broker-Badge oder über `uix` für wiederverwendbare Regeln gesetzt werden. `--uix-badge-offset-x` und `--uix-badge-offset-y` verschieben ein platziertes Badge nach dem Auflösen der Position; positive Werte bewegen es nach rechts bzw. unten.

```yaml
- type: badge
  after: "$ ha-button"
  content: 3
  variant: danger
  pill: true
  placement: bottom-end
```

Mit `for: previous` direkt nach einer `button`-Direktive erscheint ein Badge auf dem erzeugten Button:

```yaml
- type: button
  label: Wohnzimmer
  end_icon: mdi:lightbulb-fluorescent-tube-outline
  tap_action:
    action: toggle
- type: badge
  for: previous
  content: 3
  variant: danger
  pill: true
```

!!! note
    - Höchstens eines von `after` und `before` setzen.
    - `for: previous` kann nicht mit `after` oder `before` kombiniert werden.
    - Ziele vom Typ `ha-button` und `ha-tile-icon` erhalten das Badge direkt statt als Geschwisterelement.
    - `content` wird als Text eingefügt, nicht als HTML.

## Textinhalt

!!! info
    Die Direktive `text-content` ist ab 8.3.0-beta.12 verfügbar.

`text-content` fügt unmittelbar nach seinem Direktivenanker ein `<span>` mit Text ein. Das ist nützlich, wenn CSS-Pseudo-Inhalte sonst nur für ein kleines Label oder eine zweite Textzeile verwendet würden. Das erzeugte Span trägt das Attribut `data-uix-broker-text-content` und wird wiederverwendet, wenn dieselbe Direktive erneut ausgeführt wird.

```yaml
- type: text-content
  anchor: "$ div.panels-list div.wrapper ha-list-nav slot ha-list-item-button#sidebar-panel-home $ a#item div.content div.headline slot"
  content: Gesichert
  style:
    display: block
    font-size: var(--ha-font-size-s)
    font-weight: var(--ha-font-weight-medium)
    line-height: 1
    color: var(--success-color)
    width: min-content
- type: tooltip
  for: previous
  content: Alle Alarmbereiche des Hauses sind gesichert
  placement: top
```

`content` wird immer als Text eingefügt, niemals als HTML. Es akzeptiert eine Zeichenfolge oder Zahl und unterstützt erfasste Daten sowie Ergebnisse vorheriger `template`- oder `javascript`-Direktiven.

Nutze `style` für ein flaches Mapping von CSS-Eigenschaftsnamen und Zeichenfolgen- oder Zahlenwerten. Die Eigenschaften werden inline auf dem erzeugten Span gesetzt.

Wenn das Ziel ein benannter Slot ist, verwende entweder das tatsächliche `<slot>`-Element als Anker wie im Beispiel, oder verankere ein Light-DOM-Element, das diesem Slot bereits zugeordnet ist. Im zweiten Fall kopiert UIX das `slot`-Attribut des Ankers auf das erzeugte Span, damit es in denselben Slot projiziert wird. `text-content` besitzt keine eigene `slot`-Option; die Direktive folgt dem aufgelösten Anker.

| Schlüssel | Typ | Standard | Beschreibung |
| --- | --- | --- | --- |
| `content` | Zeichenfolge oder Zahl | `""` | Text, der in das erzeugte Span eingefügt wird. |
| `style` | Objekt | — | Flaches Mapping von CSS-Eigenschaftsnamen und Zeichenfolgen- oder Zahlenwerten; wird inline auf dem erzeugten Span gesetzt. |

## Tile icon

!!! info
    Die Direktive `tile-icon` ist ab 8.3.0-beta.3 verfügbar

`tile-icon` fügt neben dem Direktivenanker ein Home-Assistant-`ha-tile-icon` ein. Die Direktive verwendet dieselbe Icon-Darstellung und Aktionsbehandlung wie der [Forge tile-icon spark](../forge/sparks/tile-icon.md). Standardmäßig wird das Tile-Icon nach dem Direktivenanker eingefügt.

Nutze `after` oder `before`, um ein anderes Referenzelement auszuwählen. Diese Pfade sind relativ zum aufgelösten Direktivenanker und unterstützen die übliche UIX-`select_tree`-Syntax. Das Tile-Icon wird als Geschwisterelement des passenden Referenzelements eingefügt.

```yaml
- type: tile-icon
  entity: light.living_room
  tap_action:
    action: toggle
```

```yaml
- type: tile-icon
  anchor: "$ ha-dialog"
  before: "div.header"
  entity: light.living_room
  icon: mdi:star
  color: orange
  tap_action:
    action: more-info
```

Nutze `style` für ein flaches Mapping von CSS-Eigenschaftsnamen und Werten. Die Eigenschaften werden inline auf dem erzeugten `ha-tile-icon` gesetzt. Das ist nützlich für Positionierung und Größe, wenn Dashboard-Styling nicht an die Stelle kommt.

```yaml
- type: tile-icon
  entity: light.living_room
  style:
    margin-inline-start: 8px
    "--tile-icon-size": 28px
    z-index: 1
```

Nutze `uix` für UIX Styling, einschließlich Styles innerhalb des shadow root des Tile-Icons. Der UIX-Typ ist `broker-tile-icon`; die aufgelösten Tile-Icon-Einstellungen stehen in UIX-Templates als `config` zur Verfügung, und Ergebnisse vorheriger `template`- oder `javascript`-Direktiven stehen als `directive` bereit.

```yaml
- type: tile-icon
  entity: light.living_room
  uix:
    style: |
      :host {
        --tile-icon-size: {{ '32px' if is_state(config.entity, 'on') else '24px' }};
      }
```

| Schlüssel | Typ | Standard | Beschreibung |
| --- | --- | --- | --- |
| `after` | `string` | Direktivenanker | Relativer Selektor für das Referenzelement. Das Tile-Icon wird danach eingefügt. |
| `before` | `string` | — | Relativer Selektor für das Referenzelement. Das Tile-Icon wird davor eingefügt. |
| `entity` | `string` | — | Entity, deren Zustandsicon dargestellt wird. Sie liefert die Standard-Tap-Aktion: `toggle` für umschaltbare Entities, sonst `none`. |
| `icon` | `string` | — | MDI-Icon. Zusammen mit `entity` überschreibt es das normale Zustandsicon der Entity. |
| `icon_path` | `string` | — | SVG-Pfad, der als `iconPath` an `ha-tile-icon` übergeben wird. |
| `image_url` | `string` | — | Bild-URL, die als `imageUrl` an `ha-tile-icon` übergeben wird. |
| `color` | CSS-Farbe | — | Farbe des Tile-Icons. Mit `entity` wird sie angewendet, während die Entity aktiv ist. |
| `style` | object | — | Flaches Mapping von CSS-Eigenschaftsnamen und String- oder Zahlenwerten; wird inline auf `ha-tile-icon` gesetzt. |
| `uix` | object | — | UIX-Konfiguration, die auf das erzeugte Tile-Icon als Typ `broker-tile-icon` angewendet wird. |
| `tap_action` / `hold_action` / `double_tap_action` | action | — | Home-Assistant-Aktion, die vom Tile-Icon ausgeführt wird. |

!!! note
    - Setze höchstens eines von `after` und `before`.
    - Gib mit `icon`, `icon_path`, `image_url` oder `entity` eine Icon-Quelle an.
    - Entitybasierte Tile-Icons aktualisieren sich bei Home-Assistant-Zustandsänderungen.
    - Pointer-, Maus-, Touch- und Click-Ereignisse enden am erzeugten Icon. Dadurch reagieren Ripple oder Action-Handler eines umschließenden Elements nicht, während die eigene Aktion und Ripple des Tile-Icons erhalten bleiben.
    - Broker fügt jedem erzeugten Tile-Icon das Attribut `data-uix-broker-tile-icon` hinzu, damit es über UIX Styling ausgewählt werden kann.

## Tooltip

`tooltip` hängt ein Home-Assistant-`wa-tooltip` neben dem ausgewählten Ziel an. Die Optionen und CSS-Variablen entsprechen dem [Forge tooltip spark](../forge/sparks/tooltip.md). Standardmäßig ist `for` der aufgelöste Direktivenanker. Ein Selektor ist relativ zu diesem Anker und nutzt die normale UIX-`select_tree`-Syntax. Das Ziel muss zu einem Element aufgelöst werden, nicht zu einem terminalen shadow root.

```yaml
- type: tooltip
  content: Open the living-room light controls
  placement: bottom
```

Nutze `for: previous` direkt nach einer UI-Direktive, um den Tooltip an das gerade erzeugte Element anzuhängen. Das funktioniert derzeit mit `button` und `tile-icon` und funktioniert auch mit späteren elementerzeugenden Direktiven, ohne dass ein Elementselektor nötig ist.

```yaml
- type: button
  icon: mdi:lightbulb
  tap_action:
    action: toggle
- type: tooltip
  for: previous
  content: Toggle the light
  placement: bottom
```

```yaml
- type: tooltip
  for: "$ ha-dialog ha-icon-button"
  content: Close
  without_arrow: true
```

Nutze `style` für ein flaches Mapping von CSS-Eigenschaften. Das ist besonders nützlich, um die `--uix-tooltip-*`-Variablen direkt auf dem erzeugten Tooltip zu setzen.

```yaml
- type: tooltip
  for: previous
  content: Toggle the light
  style:
    "--uix-tooltip-background-color": var(--primary-color)
    "--uix-tooltip-content-color": white
    "--uix-tooltip-max-width": 24ch
```

`trigger` akzeptiert die durch Leerzeichen getrennten Web-Awesome-Aktivierungsmodi `hover`, `focus`, `click` und `manual`. Wenn `hover` aktiviert ist, bleibt der Tooltip offen, während sich der Mauszeiger vom Ziel in den Tooltip-Inhalt bewegt. Dadurch kann begrenzter Inhalt gescrollt werden. `manual` aktiviert nicht automatisch; nutze `open`, um den Zustand zu setzen, wenn die Direktive läuft.

```yaml
- type: tooltip
  for: previous
  trigger: manual
  open: true
  content: This tooltip is opened by the directive
```

| Schlüssel | Typ | Standard | Beschreibung |
| --- | --- | --- | --- |
| `for` | string | Direktivenanker | Zielselektor oder `previous` für die vorherige elementerzeugende Direktive. |
| `content` | string | `""` | HTML-Inhalt des Tooltip-Körpers. |
| `placement` | string | `"top"` | `top`, `top-start`, `top-end`, `bottom`, `bottom-start`, `bottom-end`, `left`, `left-start`, `left-end`, `right`, `right-start` oder `right-end`. |
| `distance` | number | `8` | Abstand in Pixeln zwischen Tooltip und Ziel. |
| `skidding` | number | `0` | Versatz in Pixeln entlang der Zielachse. |
| `show_delay` | number | `150` | Millisekunden, bevor der Tooltip angezeigt wird. |
| `hide_delay` | number | `150` | Millisekunden, bevor der Tooltip ausgeblendet wird. |
| `trigger` | string | `"hover focus"` | Durch Leerzeichen getrennte Aktivierungsmodi: `hover`, `focus`, `click` oder `manual`. |
| `open` | boolean | `false` | Setzt den offenen Zustand des Tooltips, wenn die Direktive läuft. Das ist besonders mit `trigger: manual` nützlich. |
| `without_arrow` | boolean | `false` | Blendet den Richtungspfeil aus. |
| `style` | object | — | Flaches Mapping von CSS-Eigenschaftsnamen und String- oder Zahlenwerten, inline auf `wa-tooltip` gesetzt. |

Der Tooltip wird als Geschwisterelement seines Ziels eingefügt. Setze die `--uix-tooltip-*`-CSS-Variablen auf dem Elternelement des Ziels oder einem Vorfahren, um ihn anzupassen; siehe die [CSS-Variablenreferenz des Forge tooltip spark](../forge/sparks/tooltip.md#css-variables-reference).

## Lock

`lock` legt eine Sperrfläche über den Direktivenanker und verhindert die Nutzung, bis der aktuelle Benutzer die konfigurierte PIN-, Passphrase- oder Bestätigungsabfrage abgeschlossen hat. Die Direktive verwendet dieselbe Zugriffsprüfung, Wiederholungsbehandlung, Icons und `--uix-lock-*`-CSS-Variablen wie der [Forge lock spark](../forge/sparks/lock.md).

```yaml
- type: lock
  action: tap
  duration: 5s
  entity: light.living_room
  unlocked_action:
    action: toggle
  locks:
    - code: 1234
      admins: true
```

Der Direktivenanker ist standardmäßig das gesperrte Element. Setze `for` auf einen relativen Selektor, um ein Kindelement zu sperren, oder nutze `for: previous` direkt nach einer elementerzeugenden Direktive wie `button` oder `tile-icon`.

```yaml
- type: button
  icon: mdi:account
- type: lock
  for: previous
  locks:
    - confirmation: true
      admins: true
```

Nutze `anchor`, um die Direktivenwurzel für `for`-Selektoren zu ändern. `locks`, `permissive`, `code_dialog`, `action`, `duration`, `icon_locked`, `icon_unlocked`, `icon_locked_color`, `icon_unlocked_color`, `icon_position` und `icon_size` haben dieselbe Bedeutung wie beim Forge lock spark.

`unlocked_action` ist optional. Eine normale Home-Assistant-Aktion läuft gegen `entity`; `element_tap`, `element_hold` und `element_double_tap` lösen die entsprechende Aktion aus der `config` des gesperrten Elements aus, wenn es eine besitzt.

Nutze `style` für ein flaches Mapping von CSS-Eigenschaftsnamen und Werten auf der erzeugten Sperrfläche. Die `--uix-lock-*`-CSS-Variablen sind in der Regel vorzuziehen, weil sie weiter gelten, während die Sperre zwischen gesperrten, entsperrten und blockierten Zuständen wechselt.

```yaml
- type: lock
  style:
    "--uix-lock-background": rgba(0, 0, 0, 0.25)
    "--uix-lock-icon-size": 20px
    z-index: 2
```

Nutze `uix` für UIX Styling auf der erzeugten Sperrfläche. Der UIX-Typ ist `uix-broker-lock`; die aufgelösten Lock-Einstellungen stehen als `config` zur Verfügung, und Ergebnisse früherer `template`- oder `javascript`-Direktiven stehen als `directive` bereit.

```yaml
- type: lock
  locks:
    - confirmation: true
      admins: true
  uix:
    style: |
      :host {
        --uix-lock-background: {{ 'rgba(0, 0, 0, 0.35)' if config.locks else 'transparent' }};
      }
```

## Action handler

!!! info
    Die Direktive `action-handler` ist ab 8.3.0-beta.12 verfügbar.

`action-handler` bindet den Action-Handler von Home Assistant an den Direktivenanker. Es können eine oder mehrere Standardaktionen konfiguriert werden; `tap_action`, `hold_action` und `double_tap_action` werden unterstützt. Die passende Aktion wird vom Anker als normales `hass-action`-Event ausgelöst.

Jeder konfigurierte Aktionstyp gehört UIX Broker: Sein `action`-Event erreicht keine anderen Listener auf dem Anker oder dessen Vorfahren. Verwende die Direktive daher, um vorhandenes Verhalten für diesen Aktionstyp zu ersetzen, nicht um Aktionen zu kombinieren. Einen Aktionstyp auslassen oder seine Aktion auf `none` setzen, um ihn unverändert zu lassen.

Mit `entity` wird eine Entity-ID an entitätsbasierte Aktionen wie `toggle` und `more-info` übergeben. `cursor` setzt den Cursor nur auf dem Anker dieser Direktive und ist standardmäßig `pointer`; jeder CSS-Cursorwert wie `default` oder `auto` ist möglich.

```yaml
- type: action-handler
  anchor: "$ div.menu div.title"
  tap_action:
    action: navigate
    navigation_path: /home
```

```yaml
- type: action-handler
  anchor: "$ div.menu div.title"
  cursor: default
  entity: light.living_room
  tap_action:
    action: toggle
  hold_action:
    action: more-info
  double_tap_action:
    action: navigate
    navigation_path: /dashboard-lights
```

| Schlüssel | Typ | Beschreibung |
| --- | --- | --- |
| `entity` | Zeichenfolge | Entity-ID für entitätsbasierte Aktionen. |
| `cursor` | Zeichenfolge | CSS-Cursor für den Anker. Standard ist `pointer`. |
| `tap_action` | Aktion | Beim Tippen auszuführende Aktion. |
| `hold_action` | Aktion | Beim Gedrückthalten auszuführende Aktion. |
| `double_tap_action` | Aktion | Beim Doppeltippen auszuführende Aktion. |

## Action

`action` führt einen Home-Assistant-Serviceaufruf, eine Standard-Frontend-Aktion oder eine UIX-Broker-spezifische Aktion aus.

```yaml
- type: action
  action: light.turn_on
  target:
    entity_id: light.example

- type: action
  action: fire-dom-event
  uix:
    action: toast
    data:
      message: Done
```

### JavaScript-Aktion

`action: javascript` ist eine UIX-Broker-Aktion. Lege den Code in `data.code` ab. UIX Broker übergibt automatisch `hass`, `anchor`, `event` und `captured` als Variablen. `hass` ist das aktive Home-Assistant-Objekt, `anchor` das aufgelöste DOM-Element des Interaktionsankers, `event` das auslösende Ereignis und `captured` die erfassten Daten der Interaktion.

```yaml
- type: action
  action: javascript
  data:
    code: |
      console.log(anchor, event, captured)
```

Verwende JavaScript nur aus vertrauenswürdigen UIX-Konfigurationen.

## Template

`template` rendert ein Home-Assistant-Jinja2-Template einmalig über die Template-API; es erstellt kein Template-Abonnement. Das String-Ergebnis wird für die übrigen Direktiven dieser Interaktion unter `id` gespeichert.

Jedes ungecachte Rendern ist ein Roundtrip zum Home-Assistant-Server. Vermeide dies bei Interaktionen, die häufig laufen können. Setze `cache` auf eine positive Anzahl von Millisekunden, wenn ein leicht veralteter Wert akzeptabel ist:

```yaml
- type: template
  id: example
  cache: 5000
  template: "{{ states('sensor.example') }}"
```

Der Cache liegt im Browser und wird von Template-Direktiven geteilt, die denselben Template-Text und dieselben vorherigen Direktiven-Ergebnisse verwenden. Ein gecachter Wert wird nur verwendet, wenn er jünger als die `cache`-Dauer der Direktive ist. `cache: 0` oder ein fehlendes `cache` rendert immer erneut. Der Cache speichert nur erfolgreiche Ergebnisse, wird beim Neuladen der Broker-Konfiguration geleert und beobachtet während der Cache-Dauer keine Template-Änderungen. Wenn `cache` aktiviert ist, müssen vorherige Direktiven-Ergebnisse JSON-serialisierbar sein, weil sie Teil des Cache-Schlüssels werden; zirkuläre Objekte können nicht gecacht werden.

```yaml
- type: template
  id: log_provider_url
  template: "/config/logs?provider={{ states('input_select.log_provider') }}"
- type: button
  after: "&home-assistant $ home-assistant-main $ ha-config-system-navigation $ ha-config-navigation-list $ ha-list-item-button:nth-of-type(4) $ a#item div.content"
  icon: mdi:open-in-new
  color: var(--primary-color)
  tap_action:
    action: url
    url_path: "@log_provider_url"
```

`id` muss mit einem Buchstaben oder Unterstrich beginnen und darf danach Buchstaben, Zahlen, Unterstriche und Bindestriche enthalten. Der Name `captured` ist für `@captured`-Ereignisdaten reserviert und kann nicht als ID verwendet werden. Verwende Punkt- oder Klammerpfade für Arrays, um einen gespeicherten Objekt- oder Array-Wert auszuwählen, genau wie bei `@captured`. Zitierte Klammer-Schlüssel funktionieren ebenfalls, zum Beispiel `@config_path['icon-color']` oder `@config_path["icon-color"]`.

Templates erhalten vorherige Direktiven-Ergebnisse in der Top-Level-Variable `directive`. Eine vorherige Direktive mit `id: provider` ist zum Beispiel als `{{ directive.provider }}` verfügbar. Dieser Namespace enthält nur Ergebnisse früherer Direktiven derselben Interaktion.

## JavaScript

`javascript` wertet `code` einmal aus und speichert den synchronen Rückgabewert unter `id`. Der Code erhält `hass`, `anchor`, `event`, `captured` und `directive`; `directive` enthält vorherige Direktiven-Ergebnisse derselben Interaktion. Gib einen Skalar, ein Objekt oder ein Array zurück; folgende Direktiven können den Wert ohne Konvertierung als `@id` verwenden.

```yaml
- type: javascript
  id: config_path
  code: |
    const provider = hass.states['input_select.log_provider'].state;
    return {
      path: `/config/logs?provider=${provider}`,
      label: `Open ${provider.charAt(0).toUpperCase() + provider.slice(1)} logs`,
    };
- type: button
  icon: mdi:open-in-new
  label: "@config_path.label"
  tap_action:
    action: url
    url_path: "@config_path.path"
```

Verwende JavaScript nur aus vertrauenswürdigen UIX-Konfigurationen.

## Wait

Nutze `wait`, um eine Direktivensequenz zu pausieren, ohne eine weitere Operation auszuführen. Erforderlich ist eine nicht-negative Anzahl Millisekunden.

```yaml
directives:
  - type: wait
    wait: 500
  - type: action
    action: light.turn_on
    target:
      entity_id: light.example
```

Jede Direktive akzeptiert außerdem `wait`, eine nicht-negative Anzahl Millisekunden. In dieser Form wartet UIX Broker nach Anwendung der Direktive, bevor die nächste beginnt. Eine `block`-Direktive läuft immer synchron, kann aber `wait` enthalten, um spätere Direktiven zu verzögern.

```yaml
directives:
  - type: event
    name: broker-started-event
    wait: 250
  - type: action
    action: light.turn_on
```
