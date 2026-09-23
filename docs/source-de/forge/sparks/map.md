---
description: Verwende den Map Spark, um Kartenansicht und Mittelpunkt einer Map-Karte innerhalb von UIX Forge zu bewahren.
icon: material/map
---

# :material-map: Map spark

Der `map`-Spark ergänzt eine Kartenkarte innerhalb eines mit [UIX Forge](../index.md) erstellten Elements um eine erweiterte Kartenverwaltung. Er unterstützt fünf Modi:

- **Speichermodus** (`memory: true`): Erfasst vor jedem Update Zoom und Mittelpunkt der aktuellen Karte und stellt sie danach wieder her. So bleibt die gewählte Kartenansicht erhalten. Ohne diesen Modus setzt jedes Forge-Template-Update die Karte auf den Standardzoom und die Standardposition zurück.
- **Kartenansicht anpassen** (`fit_map: true`): Passt die Kartenansicht an, wenn die Karte beim Laden nicht automatisch auf alle Entitäten zoomt, etwa in benutzerdefinierten Karten, die zunächst verborgen sind, wie `custom:auto-entities`.
- **Tourmodus** (`tour: true | object`): Bewegt die Karte automatisch zwischen mehreren Orten. Eine Pause-/Wiedergabe-Schaltfläche wird eingeblendet. Bei `tour: true` gelten die Standardwerte; mit einem Objekt lässt sich das Verhalten anpassen.
- **Zeitraumregler** (`hours_to_show: true | object`): Blendet einen interaktiven `ha-slider` ein, mit dem sich der geladene und angezeigte Zeitraum des Verlaufs in Echtzeit ändern lässt.
- **Entitätsfilter** (`entity_filter: true | object`): Blendet eine interaktive Auswahlliste ein, über die sich die auf der Karte sichtbaren Entitäten in Echtzeit ein- und ausblenden lassen.

## Basic usage

```yaml
type: custom:uix-forge
forge:
  mold: card
  sparks:
    - type: map
      memory: true
element:
  type: map
  entities:
    - device_tracker.phone
```

## Configuration

| Key | Type | Default | Description |
| --- | --- | --- | --- |
| `type` | string | — | Must be `map`. |
| `memory` | boolean | `false` | Speichert Zoomstufe und Mittelpunkt vor jeder Aktualisierung und stellt sie danach wieder her. |
| `fit_map` | boolean | `false` | Passt die Kartenansicht an alle Entitäten an, sobald die Karte sichtbar ist; nützlich bei Karten, die beim Laden verborgen sind. |
| `tour` | boolean or object | `false` | Aktiviert den Tourmodus. `true` verwendet alle Standardwerte; ein Objekt passt die Optionen an (siehe unten). |
| `hours_to_show` | boolean or object | `false` | Aktiviert den Zeitraumregler. `true` verwendet die Standardwerte; ein Objekt passt die Optionen an (siehe unten). |
| `entity_filter` | boolean or object | `false` | Aktiviert die Auswahlliste zum Filtern von Entitäten. `true` verwendet die Standardwerte; ein Objekt passt die Optionen an (siehe unten). |

### Tour sub-keys

| Key | Type | Default | Description |
| --- | --- | --- | --- |
| `period` | string or number | `10s` | Verweildauer an jedem Ort. Akzeptiert eine lesbare Zeitangabe wie `"30s"` oder `"2m"` oder eine Zahl in Millisekunden. |
| `zoom` | number | `14` | Standard-Zoomstufe beim Wechsel zu einem Ort. |
| `icon_pause` | string | `mdi:pause` | Symbol auf der eingeblendeten Schaltfläche während der Tour. |
| `icon_play` | string | `mdi:play` | Symbol auf der eingeblendeten Schaltfläche, wenn die Tour pausiert ist. |
| `icon_position` | object | `{bottom: 40px, right: 10px}` | CSS-Position der Pause-/Wiedergabe-Schaltfläche. Akzeptiert `top`, `bottom`, `left` und `right`; Zahlen werden als Pixel behandelt. |
| `poi` | list | *(nicht gesetzt)* | Liste der Orte. Fehlt sie, werden die in der `ha-map`-Karte angegebenen Entitäten verwendet. |

Each `poi` list entry may contain:

| Key | Type | Description |
| --- | --- | --- |
| `entity` | string | Entitäts-ID. Muss in der `entities`-Liste von `ha-map` enthalten sein. Breiten- und Längengrad werden aus den Zustandsattributen gelesen. |
| `latitude` | number | Breitengrad; erforderlich, wenn `entity` nicht gesetzt ist. |
| `longitude` | number | Längengrad; erforderlich, wenn `entity` nicht gesetzt ist. |
| `zoom` | number | Abweichende Zoomstufe für diesen Ort. |

### Hours to Show sub-keys

| Key | Type | Default | Description |
| --- | --- | --- | --- |
| `min` | number | `0` | Kleinster Zeitraum in Stunden, den der Regler anzeigen kann. |
| `max` | number | `24` | Größter Zeitraum in Stunden, den der Regler anzeigen kann. |
| `step` | number | `1` | Schrittweite des Reglers. |
| `position` | object | `{bottom: 40px, right: 10px}` | CSS-Position der Reglerkapsel. Akzeptiert `top`, `bottom`, `left` und `right`; Zahlen werden als Pixel behandelt. |
| `tooltip_distance` | number | `20` | Abstand des Regler-Werkzeughinweises zum Griff in Pixeln. |

Bedienelemente ohne konfigurierte Position oder mit der expliziten Position `{bottom: 40px, right: 10px}` liegen gemeinsam in einer horizontalen Reihe über der Kartenattribution. Andere Positionen, darunter `{bottom: 10px, right: 10px}`, verwenden ihre eingestellten Abstände unabhängig voneinander.

Bedienelemente ohne konfigurierte Position oder mit der expliziten Position `{bottom: 40px, right: 10px}` liegen gemeinsam in einer horizontalen Reihe über der Karten-Attribution. Alle anderen Positionen, auch `{bottom: 10px, right: 10px}`, verwenden ihre konfigurierten Abstände unabhängig voneinander.

### Entity Filter sub-keys

| Key | Type | Default | Description |
| --- | --- | --- | --- |
| `position` | object | `{bottom: 40px, right: 10px}` | CSS-Position der Filter-Schaltflächenkapsel. Akzeptiert `top`, `bottom`, `left` und `right`; Zahlen werden als Pixel behandelt. |
| `size` | string | `s` | Schaltflächengröße, zum Beispiel `s`, `m` oder `l`. |
| `variant` | string | `neutral` | Farbvariante der Schaltfläche, zum Beispiel `brand`, `neutral`, `danger`, `warning` oder `success`. |
| `appearance` | string | `filled` | Darstellung der Schaltfläche, zum Beispiel `accent`, `filled` oder `plain`. |
| `icon` | string | `mdi:filter-variant` | Startsymbol der Filterschaltfläche. |
| `label` | string | `Filter` | Beschriftung der Filterschaltfläche. Mit einer leeren Zeichenfolge wird sie ausgeblendet. |
| `group` | boolean or object | `false` | Gruppiert Entitäten nach ihrer Domäne. `true` verwendet die Standardwerte; ein Objekt legt die Gruppenbeschriftungen fest. |

#### Entity filter group sub-keys

| Key | Type | Default | Description |
| --- | --- | --- | --- |
| `persons` | string | `Persons` | Beschriftung der Entitätsgruppe für die Domäne `person`. |
| `trackers` | string | `Trackers` | Beschriftung der Entitätsgruppe für die Domäne `device_tracker`. |
| `zones` | string | `Zones` | Beschriftung der Entitätsgruppe für die Domäne `zone`. |

### CSS-Variablen des Tourmodus

Die Pause-/Wiedergabeschaltfläche lässt sich über CSS-Variablen auf `ha-card` oder einem übergeordneten Element gestalten:

| Variable | Default | Description |
| --- | --- | --- |
| `--uix-map-tour-icon-color` | `var(--primary-color)` | Icon color. |
| `--uix-map-tour-icon-ring-color` | `var(--uix-map-tour-icon-color)` | Countdown ring color (defaults to icon color). |
| `--uix-map-tour-icon-background` | `rgba(255,255,255,0.8)` | Button background. |
| `--uix-map-tour-icon-box-shadow` | `0 1px 5px rgba(0,0,0,0.4)` | Box shadow of the icon container. |
| `--uix-map-tour-icon-width` | `auto` | Button width. |
| `--uix-map-tour-icon-height` | `auto` | Button height. |
| `--uix-map-tour-icon-border-radius` | `9999px` | Button border radius (pill by default). |
| `--uix-map-tour-icon-z-index` | `1000` | Button z-index (Leaflet controls use 1000). |

### CSS-Variablen des Zeitraumreglers

Der Verlaufsregler lässt sich über CSS-Variablen auf `ha-card` oder einem übergeordneten Element gestalten:

| Variable | Default | Description |
| --- | --- | --- |
| `--uix-map-slider-background` | `rgba(255,255,255,0.8)` | Background color of slider container. |
| `--uix-map-slider-text-color` | `var(--primary-text-color, #212121)` | Color of the duration label next to the slider. |
| `--uix-map-slider-width` | `100px` | Explicit width of the slider component. |
| `--uix-map-slider-border-radius` | `9999px` | Slider container border radius (pill by default). |
| `--uix-map-slider-padding` | `4px 12px` | Padding of the capsule container. |
| `--uix-map-slider-box-shadow` | `0 1px 5px rgba(0,0,0,0.4)` | Box shadow of the container element. |
| `--uix-map-slider-z-index` | `1000` | Controls overlay depth of the slider capsule. |
| `--uix-map-slider-label-min-width` | `28px` | Minimum width of the duration label. |
| `--uix-map-slider-thumb-size` | *(unset)* | Height and width of the slider thumb. |
| `--uix-map-slider-thumb-height` | `16px` | Height of the slider thumb component structure (defaults to thumb-size if set). |
| `--uix-map-slider-thumb-width` | `16px` | Width of the slider thumb component structure (defaults to thumb-size if set). |
| `--uix-map-slider-track-size` | `4px` | Thickness of the slider track. |
| `--uix-map-slider-track-color` | `var(--disabled-color)` | Background base track color. |
| `--uix-map-slider-indicator-color` | `var(--primary-color)` | Color of the active indicator bar. |
| `--uix-map-slider-thumb-color` | `var(--uix-map-slider-indicator-color)` | Color of the circular slider thumb. |
| `--uix-map-slider-thumb-hover-opacity` | `0.08` | Hover opacity surrounding the thumb. |
| `--uix-map-slider-thumb-pressed-opacity` | `0.12` | Opacity of thumb halo while active/pressed. |
| `--uix-map-slider-thumb-box-shadow` | `inherit` | Custom shadow styling applied to the interactive thumb item. |
| `--uix-map-slider-tooltip-color` | `var(--primary-text-color)` | Text color inside the popup thumb tooltip. |
| `--uix-map-slider-tooltip-font-size` | `var(--ha-font-size-s)` | Font size of text inside the tooltip. |
| `--uix-map-slider-tooltip-font-weight` | `var(--ha-font-weight-normal)` | Font thickness of elements inside tooltip. |
| `--uix-map-slider-tooltip-background-color` | `var(--secondary-background-color)` | Background color of popup tooltip bubble. |
| `--uix-map-slider-tooltip-border-radius` | `var(--ha-border-radius-sm)` | Corner rounding of tooltip bubble. |
| `--uix-map-slider-tooltip-border-width` | `0px` | Border thickness of tooltip. |
| `--uix-map-slider-tooltip-border-color` | `currentColor` | Border color of tooltip bubble. |
| `--uix-map-slider-tooltip-border-style` | `none` | Border line style. |

### CSS-Variablen des Entitätsfilters

Das Dropdown des Entitätsfilters lässt sich über CSS-Variablen auf `ha-card` oder einem übergeordneten Element gestalten:

| Variable | Default | Description |
| --- | --- | --- |
| `--uix-map-entity-filter-background` | `rgba(255,255,255,0.8)` | Background color of entity filter container. |
| `--uix-map-entity-filter-padding` | `4px` | Padding of the filter capsule container. |
| `--uix-map-entity-filter-border-radius` | `9999px` | Entity filter container border radius (pill by default). |
| `--uix-map-entity-filter-box-shadow` | `0 1px 5px rgba(0,0,0,0.4)` | Box shadow of the container element. |
| `--uix-map-entity-filter-z-index` | `1000` | Controls overlay depth of the filter capsule. |
| `--uix-map-entity-filter-dropdown-min-width` | `180px` | Minimum width of the opened dropdown menu list. |
| `--uix-map-entity-filter-item-icon-color` | `var(--ha-color-fill-neutral-loud-resting)` | Color of the check icon of entity filter list items. |
| `--uix-map-entity-filter-item-icon-checked-color` | `var(--uix-map-entity-filter-item-icon-color, var(--primary-color))` | Color of the check icon of entity filter items when checked. |

## How it works

**Speichermodus:**

Vor jeder Aktualisierung des erstellten Elements durch eine Forge-Vorlage führt der Spark folgende Schritte aus:

1. Liest `zoom` und `center` der aktuellen Karten-Engine innerhalb von `ha-map`.
2. Wartet, bis das erstellte Element und anschließend `ha-map` ihre Aktualisierung abgeschlossen haben.
3. Stellt die gespeicherte Position über die Karten-Engine von Home Assistant wieder her, ohne eine Animation auszulösen.

Ist die Karten-Engine beim Aktualisieren noch nicht initialisiert, etwa beim ersten Rendern, wird das Speichern übersprungen und nichts wiederhergestellt. Beim ersten Laden zeigt die Karte dann ihre Standardansicht.

**Kartenansicht anpassen:**

Sobald das geschmiedete Element und `ha-map` ihr Update beendet haben und die Karten-Engine eine nutzbare Größe hat, ruft der Spark `fitMap()` auf `ha-map` auf.

**Tourmodus:**

Sobald die Karte bereit ist (und gegebenenfalls `fit_map` abgeschlossen wurde), führt der Spark folgende Schritte aus:

1. Ermittelt die Ortsliste aus der `poi`-Konfiguration oder aus den Zustandsattributen `latitude` und `longitude` der `ha-map`-Entitäten.
2. Fügt ein `ha-icon-button`-Overlay mit einem kreisförmigen SVG-Countdown-Ring in den Container der Karten-Engine ein.
3. Bewegt die Karte sofort zum ersten POI und startet dann einen wiederkehrenden Timer, der über die Karten-Engine alle `period` Sekunden zum nächsten POI wechselt.
4. Der Countdown-Ring leert sich während jedes Zeitraums und zeigt so die verbleibende Zeit am aktuellen Ort an.
5. Beim Betätigen der Pause-/Wiedergabeschaltfläche wird der Timer angehalten oder neu gestartet und der Ring ausgeblendet oder zurückgesetzt.

Wenn `memory: true` und `tour` gleichzeitig aktiv sind, werden Wiederherstellungen bei hass-Aktualisierungen während der laufenden Tour unterdrückt, damit die Animation nicht unterbrochen wird.

**Zeitraumregler:**

Ist der Modus aktiv, führt der Spark folgende Schritte aus:

1. Zeigt einen horizontalen `ha-slider` in einem kapselartigen Overlay an.
2. Ist zusätzlich `tour` mit der Standardposition aktiv, wird der Regler automatisch nach links verschoben, damit sich die Bedienelemente nicht überlagern.
3. Setzt und begrenzt `hui-map-card` `_config.hours_to_show` anhand der Reglerbewegungen und lädt Verlaufsdaten in Echtzeit.
4. Behält den ausgewählten Wert bei erneuten, vorlagenbedingten Forge-Darstellungen bei.

**Entitätsfilter:**

Ist der Modus aktiv, führt der Spark folgende Schritte aus:

1. Blendet ein Dropdown mit `ha-dropdown` und einer `ha-button` als Auslöser ein.
2. Ermittelt die Kartenentitäten und zeigt sie mit ihrem Anzeigenamen als Kontrollkästchen an.
3. Filtert die sichtbaren Entitäten direkt. Da eine Kartenkarte ohne Entitäten einen Fehler auslöst, lässt sich die letzte ausgewählte Entität nicht abwählen.
4. Ist `show_all: true` in der Forge-Kartenkonfiguration gesetzt, erscheint im Dropdown auch die Option `Show All`. Solange ein Filter aktiv ist, werden neue Kartenentitäten erst nach Auswahl von `Show All` angezeigt.
5. Ist zusätzlich `tour` oder `hours_to_show` mit der Standardposition aktiv, wird der Regler automatisch nach links verschoben, um Überlagerungen zu vermeiden.
6. Ist auch `tour` aktiv, startet eine Änderung der gefilterten Entitäten die Kartentour neu.

!!! note
    Der Spark greift auf `hui-map-card` innerhalb des erstellten Elements und auf `ha-map` in dessen Shadow Root zu. Er unterstützt die Karten-Engine von Home Assistant (MapLibre, sofern verfügbar, andernfalls Leaflet) sowie ältere Leaflet-basierte Frontends. Ist das erstellte Element keine Kartenkarte oder in ein Element eingebettet, das `hui-map-card` nicht bereitstellt, haben die Modi keine Wirkung.

## Examples

### Kartenansicht mit auto-entities anpassen

Bei einer Kartenkarte in `custom:auto-entities` kann die Karte wegen des anfänglichen Ausblendens beim Laden nicht automatisch angepasst werden. Der Modus zum Anpassen der Kartenansicht sorgt in diesem Fall dafür, dass sie beim ersten Anzeigen passend skaliert wird.

Der Kürze halber enthält das Beispiel keine Include-Filter.

```yaml
type: custom:auto-entities
entities:
  - zone.london
filter:
  include: []
  exclude: []
card:
  type: custom:uix-forge
  forge:
    mold: card
    sparks:
      - type: map
        fit_map: true
  element:
    type: map
    fit_zones: true
    uix:
      style: |
        :host {
          display: block;
          height: 400px;
        }
```

Without `fit_map: true`:

![Forged map without map spark fit_map true](../../assets/page-assets/forge/sparks/map-auto-entities-no-spark.png)

With `fit_map: true`:

![Forged map with map spark fit_map true](../../assets/page-assets/forge/sparks/map-auto-entities.png)

### Tour mode with default settings

Automatically cycle through all map entities using defaults (10 s per stop, pause/play button in bottom-right corner):

```yaml
type: custom:uix-forge
forge:
  mold: card
  sparks:
    - type: map
      tour: true
element:
  type: map
  entities:
    - device_tracker.my_phone
    - device_tracker.my_tablet
```

:material-movie: [Map spark tour mode example animation (mp4)](../../assets/page-assets/forge/sparks/map-tour.mp4){ data-type="video" class="glightbox" }

### Tour mode with custom POI list

Fly between fixed coordinates and specific entities with individual zoom levels:

```yaml
type: custom:uix-forge
forge:
  mold: card
  sparks:
    - type: map
      tour:
        period: 15s
        zoom: 13
        icon_pause: mdi:pause-circle
        icon_play: mdi:play-circle
        icon_position:
          bottom: 16px
          left: 16px
        poi:
          - latitude: 51.614387 
            longitude: -0.731585
            zoom: 11
          - entity: device_tracker.my_tablet
            zoom: 14
element:
  type: map
  entities:
    - device_tracker.my_phone
    - device_tracker.my_tablet
```

:material-movie: [Map spark tour mode with pois example animation (mp4)](../../assets/page-assets/forge/sparks/map-tour-pois.mp4){ data-type="video" class="glightbox" }

### Styling the tour button

Override the default pill shape with rounded corners and a dark background:

```yaml
type: custom:uix-forge
forge:
  mold: card
  sparks:
    - type: map
      tour: true
element:
  type: map
  entities:
    - device_tracker.phone
  uix:
    style: |
      ha-card {
        --uix-map-tour-icon-color: white;
        --uix-map-tour-icon-background: rgba(255,0,0,0.5);
        --uix-map-tour-icon-border-radius: 4px;
      }
```

![Map spark styling example](../../assets/page-assets/forge/sparks/map-tour-style.png)

### Hours to show history slider

Enable a customizable history duration slider to load between 1 and 48 hours of tracker data on the map:

```yaml
type: custom:uix-forge
forge:
  mold: card
  sparks:
    - type: map
      hours_to_show:
        min: 1
        max: 48
        step: 2
        position:
          bottom: 15px
          left: 15px
element:
  type: map
  hours_to_show: 3
  default_zoom: 8
  entities:
    - device_tracker.phone
```

![Map spark hours to show example](../../assets/page-assets/forge/sparks/map-hours-to-show.png)

### Entity selection filter dropdown

Enable an interactive dropdown to toggle entity tracks visibility in real-time:

```yaml
type: custom:uix-forge
forge:
  mold: card
  sparks:
    - type: map
      entity_filter: true
element:
  type: map
  entities:
    - device_tracker.phone
    - device_tracker.tablet
```

![Map spark entity filter example](../../assets/page-assets/forge/sparks/map-entity-filter.png)

Customise the trigger button text, size, color and icon styling:

```yaml
type: custom:uix-forge
forge:
  mold: card
  sparks:
    - type: map
      entity_filter:
        label: "Trackers"
        icon: "mdi:account-multiple"
        size: "m"
        variant: "brand"
        appearance: "filled"
element:
  type: map
  entities:
    - device_tracker.phone
    - device_tracker.tablet
```

![Map spark entity filter with style example](../../assets/page-assets/forge/sparks/map-entity-filter-style.png)

Display entity filter grouped by domain with custom labels

```yaml
type: custom:uix-forge
forge:
  mold: card
  sparks:
    - type: map
      entity_filter:
        group:
          persons: Household
          trackers: Phones
          zones: Places
element:
  type: map
  entities:
    - zone.london
    - device_tracker.phone
    - device_tracker.tablet
```

![Map spark entity filter with groups example](../../assets/page-assets/forge/sparks/map-entity-filter-groups.png)
