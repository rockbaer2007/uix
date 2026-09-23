---
title: UIX Forge
description: Erfahre mehr über UIX Forge, ein leistungsfähiges benutzerdefiniertes Element, das Vorlagen, Sparks und UIX-Styling verbindet.
---
Mit UIX Forge lassen sich Home-Assistant-Elemente erstellen, deren gesamte Konfiguration Vorlagen verwenden kann. Zusätzlich können die Elemente mit [UIX-Forge-Sparks](./sparks/) erweitert werden.

Unterstützt werden Home-Assistant-Karten, Badges, Zeilen, Abschnitte und Picture-Elemente. Kontextübergreifende Molds ermöglichen es, einen Elementtyp in einem anderen übergeordneten Kontext einzubetten, zum Beispiel eine Karte als Zeile in einer Entities-Karte. Siehe [kontextübergreifende Molds](./forge.md#cross-context-molds).

Die vollständige Forge-Konfigurationsreferenz findest du unter [Forge](./forge.md).

## Foundries

Eine **Foundry** ist eine auf dem Server gespeicherte UIX-Forge-Vorlage. Damit kannst du wiederverwendbare `forge`-, `element`- und `uix`-Konfigurationen einmal definieren und in mehreren Karten verwenden. Binde eine Foundry mit dem Schlüssel `foundry:` ein und überschreibe lokal nur die Einstellungen, die du ändern möchtest.

Die Anleitung zu [Foundries](./foundries.md) erklärt das Zusammenführen, verschachtelte Foundries und die Verwaltung über die Integrationsoptionen.

## Sparks

Sparks sind optionale Funktionen, die du zur Liste `forge.sparks` hinzufügst. Jeder Spark besitzt einen Schlüssel `type` und eigene Optionen.

Verfügbare Sparks:

- :speech_balloon: [Tooltip](./sparks/tooltip.md) — fügt einem Element im erstellten Element einen gestalteten Tooltip hinzu.
- :material-button-cursor: [Schaltfläche](./sparks/button.md) - fügt vor oder nach einem Element eine gestaltete Schaltfläche (`ha-button`) mit Aktionen ein.
- :label: [Attribut](./sparks/attribute.md) — fügt ein Attribut hinzu, ersetzt oder entfernt es bei einem Element.
- :zap: [Ereignis](./sparks/event.md) — empfängt DOM-Ereignisse von `fire-dom-event`-Aktionen und stellt deren Daten als Vorlagenvariablen bereit.
- :star: [Tile-Symbol](./sparks/tile-icon.md) — fügt vor oder nach einem Element ein `ha-tile-icon`-Element ein.
- :shield: [Status-Badge](./sparks/state-badge.md) - fügt vor oder nach einem Element ein `state-badge`-Element ein.
- :material-grid: [Raster](./sparks/grid.md) - wendet ein **CSS-Grid**-Layout auf einen Container innerhalb eines erstellten Elements an.
- :mag: [Suche](./sparks/search.md) - durchsucht einen Container anhand eines CSS-Selektors und optionalen Textes und verändert die gefundenen Elemente.
- :material-map: [Map](./sparks/map.md) — bewahrt die Kartenansicht und ergänzt Touren, Verlaufsregler und Entitätsfilter mit konfigurierbaren Positionen für die Bedienelemente.
- :material-lock: [Sperre](./sparks/lock.md) — legt ein Schlosssymbol über ein Element und blockiert die Interaktion, bis eine PIN, Passphrase oder Bestätigung eingegeben wurde.
- :material-star-four-points-outline: [Overlay-Symbol](./sparks/overlay-icon.md) — legt ein `ha-icon` oder `ha-state-icon` über ein Element.
- :material-image-outline: [Hintergrund](./sparks/background.md) — fügt hinter einem Element eine Hintergrundebene mit Farbe, Bild, Video oder Live-Kamera ein.
- :material-palette: [Theme](./sparks/theme.md) — wendet ein Frontend-Theme auf das erstellte Element oder eines seiner untergeordneten Elemente an.
