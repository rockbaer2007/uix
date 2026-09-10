---
title: Interaktionsanker
description: Wähle das Element aus, das UIX-Broker-Regeln und -Direktiven standardmäßig verwenden.
---
# Interaktionsanker

Ein Interaktionsanker wählt das Element aus, das Host-Element-Regeln prüfen und Direktiven standardmäßig verwenden. Browser- und Shortcut-Interaktionen können aus dem zusammengesetzten Pfad des auslösenden Ereignisses auswählen oder einen UIX-`select_tree`-Pfad verwenden. Server-Interaktionen verwenden ausschließlich `select_tree`-Pfade.

Für knappes YAML wird `anchor` meist in kompakter Form geschrieben. Die folgende Tabelle fasst zusammen, wann Event-Path-Auswahl und wann `select_tree` verwendet wird.

| `anchor:` | Methode |
| --- | --- |
| `target` | Verwendet [Event-Path](#event-path-anker)-Auswahl und löst auf das Ziel eines `browser`- oder `shortcut`-[Realm](realms.md)-Ereignisses auf. Im `server`-Realm ist dies nicht verfügbar. |
| `<`, `<$`, `<selector> <$` oder `<selector> <$$` | Verwendet [Event-Path](#event-path-anker)-Auswahl. Im `server`-Realm ist dies nicht verfügbar. |
| Eine Zeichenfolge, die mit `&` beginnt | Verwendet `select_tree`-Auswahl ab `document`. In allen [Realms](realms.md) verfügbar. |
| `{ select_tree: <path> }` | Verwendet die Langform der `select_tree`-Auswahl ab `document`. In allen Realms verfügbar. |

## Event-Path-Anker

Event-Path-Ausdrücke werden von rechts nach links ab dem impliziten `target` ausgewertet. `target` ist dabei das innerste Element, das von [`event.composedPath()`](https://developer.mozilla.org/en-US/docs/Web/API/Event/composedPath) zurückgegeben wird.

| Anker | Ergebnis |
| --- | --- |
| `target` | Das ursprüngliche innerste Ereignisziel. In allen anderen Formen ist `target` aus Gründen der Kürze optional, kann aber zur besseren Lesbarkeit angegeben werden. |
| `< [target]` | Das Elternelement von `target`. |
| `<$ [target]` | Der erste shadow-root-Host oberhalb von `target`. |
| `<selector> <$ [target]` | Das erste passende Element im Light DOM dieses ersten shadow hosts. |
| `<selector> <$$ [target]` | Das erste passende Element beim nach außen gehenden Durchlaufen des zusammengesetzten Pfads, inklusive shadow-root-Grenzen. |

!!! note
    Event-Path-Interaktionsanker werden synchron aufgelöst und können mit der [`block`-Direktive](directives.md#block) verwendet werden.

```yaml
# Nächster shadow host des Ereignisziels
anchor: "<$"

# Ein passendes ha-automation-row-Element im Light DOM des Ziel-Hosts
anchor: "ha-automation-row <$"

# Das erste passende ha-automation-row-Element im nach außen gehenden zusammengesetzten Pfad,
# inklusive shadow-root-Grenzen
anchor: "ha-automation-row <$$"
```

`<` und `<$` akzeptieren `target` explizit rechts und können zur besseren Lesbarkeit angegeben werden; kürzer ist es jedoch, es wegzulassen. `<$$` benötigt einen nach außen gerichteten Selektor.

## Select-tree-Anker

Nutze einen normalen UIX-`select_tree`-Pfad, wenn ein Anker nicht durch den Ereignispfad bestimmt wird. Das gilt auch für jede Server-Interaktion.

!!! note
    `select_tree`-Pfade werden ausführlich unter [DOM-Navigation](../concepts/dom.md) beschrieben.

```yaml
# Kompakte absolute Form
anchor: "&home-assistant $ hui-dialog-create-card"

# Langform, immer absolut ab document, ohne dass `&` nötig ist
anchor:
  select_tree: "home-assistant $ home-assistant-main $ ha-panel-lovelace $ hui-root"
```

Für Interaktionen ohne `block` versucht UIX Broker einen fehlenden `select_tree`-Anker alle 50 ms bis zu zwei Sekunden lang erneut aufzulösen. Das ist nützlich für Oberflächen wie Dialoge, die erst nach dem auslösenden Ereignis gemountet werden.

## Anker in Regeln und Direktiven

Regeln und Direktiven verwenden standardmäßig den Interaktionsanker. Sie können den Interaktionsanker aber mit einer eigenen `anchor`-Konfiguration überschreiben, entweder in kompakter relativer oder absoluter Form oder in der langen `select_tree`-Form. Bei der kompakten absoluten Form beginnt der Pfad mit `&`.

Relativ:

```yaml
- type: property
  # Anker der property-Direktive relativ zum Interaktionsanker (in diesem Beispiel ein Dialog)
  anchor: "$ ha-dialog div.body hui-card-picker $ div#content>ha-expansion-panel:nth-of-type(1)"
  set: expanded
  value: false
```

Absolute Kurzform:

```yaml
- type: call
  anchor: "&home-assistant $ ha-more-info-dialog"
  method: closeDialog
```

Absolute Langform:

```yaml
- type: call
  anchor:
    select_tree: "home-assistant $ ha-more-info-dialog"
  method: closeDialog
```

## Pfade in der Browser-Konsole finden

Um einen absoluten Ankerpfad zu finden, wähle im Browser-Inspector ein Element aus und führe aus:

```javascript
uix_broker_absolute_path($0)
```

Dies gibt einen kompakten absoluten Interaktionsankerpfad aus, der mit `&` beginnt.

Für einen Direktiven- oder Regelanker relativ zu einem zuvor aufgelösten Interaktionsanker verwendest du:

```javascript
uix_broker_path($0)
```

Der Helfer wählt den nächstliegenden passenden aktuellen Interaktionsanker. Übergib den Anker explizit als zweiten Parameter, wenn mehrere Interaktionen überlappen:

```javascript
uix_broker_path($0, $1)
```

Details zu den Konsolenhelfern findest du unter [DOM-Inspektionshelfer](../concepts/dom.md#uix_broker_path0-broker-directive-anchor-helper).
