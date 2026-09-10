---
title: Beispiele
description: UIX-Broker-Beispiele
---
# Beispiele

## Card-Tab im UI-Dialog zum Hinzufügen von Cards standardmäßig auswählen

Ergebnis:

- Der Card-Tab wird ausgewählt, wenn sich der Dialog zum Hinzufügen einer Card öffnet.
- Der erste Expander, Suggested oder Favorites, ist geöffnet.
- Alle anderen Expander werden geschlossen.

Vorgehen:

<!-- markdownlint-configure-file {"MD007": { "indent": 4 }} -->
- Auf `show-dialog` im `browser`-Realm lauschen.
- Die passende Regel greift nur, wenn das `show-dialog`-Ereignis den `dialogTag` `hui-dialog-create-card` hat.
- Einen absoluten Interaktionsanker in Kurzform verwenden, der den Dialog (`hui-dialog-create-card`) findet.
- Direktiven:
    - Die Eigenschaft `_currTab` des Dialogs auf `card` setzen. Da diese Eigenschaft reaktiv ist, muss kein Update erzwungen werden.
    - Die Eigenschaft `expanded` des ersten Expanders mit einem relativen Kurzform-Anker auf `true` setzen.
    - Die Eigenschaft `expanded` der anderen Expander mit relativen Kurzform-Ankern auf `false` setzen.

```yaml
uix_broker:
  - realm: browser
    listen: show-dialog
    anchor: '&home-assistant $ hui-dialog-create-card'

    rules:
      - '@captured.dialogTag': hui-dialog-create-card
    directives:
      - type: property
        set: _currTab
        value: card
        wait: 1000
      - type: property
        anchor: >-
          $ ha-dialog div.body hui-card-picker $ div#content div:nth-of-type(1)
          ha-expansion-panel
        set: expanded
        value: true
      - type: property
        anchor: >-
          $ ha-dialog div.body hui-card-picker $
          div#content>ha-expansion-panel:nth-of-type(1)
        set: expanded
        value: false
      - type: property
        anchor: >-
          $ ha-dialog div.body hui-card-picker $
          div#content>ha-expansion-panel:nth-of-type(2)
        set: expanded
        value: false
      - type: property
        anchor: >-
          $ ha-dialog div.body hui-card-picker $
          div#content>ha-expansion-panel:nth-of-type(3)
        set: expanded
        value: false
```

!!! tip
    Speichere das YAML als neue Datei in deinem Home-Assistant-Konfigurationsverzeichnis oder einem Unterverzeichnis und registriere sie anschließend über den UIX-Options-Konfigurationsflow.

## Automation-Sidebar und YAML-Modus

### Automation-Editor-Sidebar standardmäßig im YAML-Modus öffnen

Kombiniere dieses Beispiel mit dem folgenden Beispiel, um das Umschalten des YAML-Modus zu erlauben. Für sich allein sperrt dieses Beispiel die Automation-Sidebar darauf, **immer** den YAML-Modus zu verwenden.

Ergebnis:

- Die Automation-Sidebar wird in den YAML-Modus gesetzt.
- In der Kopfzeile der Automation-Sidebar wird eine Schaltfläche zum Umschalten des YAML-Modus hinzugefügt.

Vorgehen:

- Auf das Ereignis `open-sidebar` im `browser`-Realm lauschen.
- `reentrant: false` setzen, um erneuten Eintritt zu verhindern, wenn das Umschalten des YAML-Modus selbst `open-sidebar` auslöst.
- Der Interaktionsanker ist `manual-automation-editor`; er wird durch eine nach außen gehende Suche über den zusammengesetzten Ereignispfad und shadow-root-Grenzen gefunden.
- Eine Host-Element-Pfadauswahlregel in Kurzform verwenden, damit die Interaktion nur fortgesetzt wird, wenn `uixBlockAutoYamlMode` auf dem JavaScript-Objekt des Interaktionsankers **nicht** existiert. Das ist wichtig, wenn dieses Beispiel mit dem folgenden kombiniert wird.
- Eine `call`-Direktive verwenden, um `_toggleYamlMode()` auf `ha-automation-sidebar` aufzurufen. Dieses Element wird durch Suche im ersten shadow root des Interaktionsankers `manual-automation-editor` aufgelöst.
- Eine `button`-Direktive verwenden, um vor dem Drei-Punkte-Menü der Sidebar eine Schaltfläche zu platzieren. Die Aktion nutzt die UIX-`event`-Aktion, in die UIX Broker das Ankerelement injiziert. Dadurch kann `toggle-yaml-mode` durch `manual-automation-editor` nach oben bubblen, sodass das nächste Beispiel [YAML-Modus im Automation-Editor umschalten erlauben](#yaml-modus-im-automation-editor-umschalten-erlauben) sowohl die normale Umschaltfläche im Dropdown als auch die hinzugefügte UIX-Broker-Schaltfläche abdeckt.

```yaml
  - realm: browser
    listen: open-sidebar
    reentrant: false
    anchor: manual-automation-editor <$$ target
    rules:
      - '{!.uixBlockAutoYamlMode}'
      - anchor: $ ha-automation-sidebar $$ ha-automation-sidebar-card
        match: '{.yamlMode=false}'
    directives:
      - anchor: $ ha-automation-sidebar
        method: _toggleYamlMode
        type: call
      - type: button
        before: $ ha-automation-sidebar $$ ha-automation-sidebar-card $ ha-dialog-header slot:nth-of-type(3) ha-dropdown
        icon: mdi:code-braces
        tap_action:
          action: fire-dom-event
          uix:
            action: event
            name: toggle-yaml-mode
```

### YAML-Modus im Automation-Editor umschalten erlauben

Nutze dieses Beispiel zusammen mit dem vorherigen und dem nächsten Beispiel, das ein Tastenkürzel zum Umschalten des YAML-Modus hinzufügt.

Der Menüeintrag „Toggle YAML Mode“ im Automation-Editor führt Code aus, der `toggle-yaml-mode` auslöst. Am Ende ruft dies `_toggleYamlMode()` auf `manual-automation-editor` auf. Ohne Koordination würde dadurch immer der YAML-Modus erzwungen. Dieses Beispiel umgeht das, indem es das Ereignis blockiert, eine Guard-Eigenschaft setzt, die Funktion direkt aufruft und die Guard-Eigenschaft anschließend wieder löscht.

Ergebnis:

- Der Menüeintrag „Toggle YAML Mode“ kann zwischen visuellem Editor und YAML-Modus umschalten.

Vorgehen:

- Auf `toggle-yaml-mode` im `browser`-Realm lauschen.
- Der Interaktionsanker ist `manual-automation-editor`; er wird durch eine nach außen gehende Suche über den zusammengesetzten Ereignispfad und shadow-root-Grenzen gefunden.
- Direktiven:
    - Das Ereignis blockieren, weil es direkt behandelt wird.
    - `uixBlockAutoYamlMode` auf `manual-automation-editor` auf `true` setzen.
    - Eine `call`-Direktive verwenden, um `_toggleYamlMode()` auf `ha-automation-sidebar` aufzurufen. Da das vorherige Beispiel prüft, ob `uixBlockAutoYamlMode` fehlt, führt es seine Direktive zum Erzwingen des YAML-Modus nicht aus.
    - `uixBlockAutoYamlMode` löschen, sodass das vorherige Beispiel beim Öffnen der Sidebar wieder den YAML-Modus erzwingt.

```yaml
  - realm: browser
    listen: toggle-yaml-mode
    anchor: manual-automation-editor <$$ target
    directives:
      - type: block
      - type: property
        set: uixBlockAutoYamlMode
        value: true
      - anchor: $ ha-automation-sidebar
        method: _toggleYamlMode
        type: call
      - type: property
        clear: uixBlockAutoYamlMode
```

### YAML-Modus im Automation-Editor per Tastenkürzel umschalten

Ergebnis:

- Ein Tastenkürzel schaltet den YAML-Modus im Automation-Editor um.

Vorgehen:

- Auf ein Tastenkürzel im `shortcut`-Realm lauschen (`$mod+Shift+Y` im folgenden Code; passe es nach Bedarf an).
- Den absoluten Interaktionsanker `&home-assistant $$ manual-automation-editor` verwenden, weil das Ziel des Tastenkürzels jedes DOM-Element sein kann.
- Direktiven:
    - `uixBlockAutoYamlMode` auf `manual-automation-editor` auf `true` setzen.
    - Eine `call`-Direktive verwenden, um `_toggleYamlMode()` auf `ha-automation-sidebar` aufzurufen. Da das automatische YAML-Modus-Beispiel prüft, ob `uixBlockAutoYamlMode` fehlt, erzwingt es den YAML-Modus nicht.
    - `uixBlockAutoYamlMode` löschen, sodass das automatische YAML-Modus-Beispiel beim Öffnen der Sidebar wieder den YAML-Modus erzwingt.

```yaml
  - realm: shortcut
    enabled: true
    listen: $mod+Shift+Y
    anchor: '&home-assistant $$ manual-automation-editor'
    directives:
      - type: property
        set: uixBlockAutoYamlMode
        value: true
      - anchor: $ ha-automation-sidebar
        method: _toggleYamlMode
        type: call
      - type: property
        clear: uixBlockAutoYamlMode
```

## Automation-Sidebar und YAML-Modus vollständig

??? example "Vollständiges YAML für die drei Automation-Sidebar-Beispiele"
    Speichere das YAML als neue Datei in deinem Home-Assistant-Konfigurationsverzeichnis oder einem Unterverzeichnis und registriere sie anschließend über den UIX-Options-Konfigurationsflow.
    ```yaml
    uix_broker:
      - realm: browser
        listen: open-sidebar
        reentrant: false
        anchor: manual-automation-editor <$$ target
        rules:
          - '{!.uixBlockAutoYamlMode}'
          - anchor: $ ha-automation-sidebar $$ ha-automation-sidebar-card
            match: '{.yamlMode=false}'
        directives:
          - anchor: $ ha-automation-sidebar
            method: _toggleYamlMode
            type: call
          - type: button
            before: $ ha-automation-sidebar $$ ha-automation-sidebar-card $ ha-dialog-header slot:nth-of-type(3) ha-dropdown
            icon: mdi:code-braces
            tap_action:
              action: fire-dom-event
              uix:
                action: event
                name: toggle-yaml-mode
      - realm: browser
        listen: toggle-yaml-mode
        anchor: manual-automation-editor <$$ target
        directives:
          - type: block
          - type: property
            set: uixBlockAutoYamlMode
            value: true
          - anchor: $ ha-automation-sidebar
            method: _toggleYamlMode
            type: call
          - type: property
            clear: uixBlockAutoYamlMode
      - realm: shortcut
        enabled: true
        listen: $mod+Shift+Y
        anchor: '&home-assistant $$ manual-automation-editor'
        directives:
          - type: property
            set: uixBlockAutoYamlMode
            value: true
          - anchor: $ ha-automation-sidebar
            method: _toggleYamlMode
            type: call
          - type: property
            clear: uixBlockAutoYamlMode
    ```

## Entity-Trigger beim Hinzufügen eines Automation-Editor-Elements priorisieren

Ergebnis:

- Beim Hinzufügen eines Automation-Editor-Elements wird direkt zu Entity-Triggern gesprungen.

Vorgehen:

- Auf das Ereignis `show-dialog` im `browser`-Realm lauschen.
- Die erste passende Regel greift nur, wenn das `show-dialog`-Ereignis den `dialogTag` `add-automation-element-dialog` hat.
- Die zweite passende Regel greift nur, wenn der Typ `trigger` ist, nicht bei anderen Typen wie `action` oder `condition`.
- Einen absoluten Interaktionsanker in Kurzform verwenden, der den Dialog (`add-automation-element-dialog`) findet.
- Direktiven:
    - Die Eigenschaft `_tab` auf `groups` setzen; `groups` ist der Wert für „By Type“.
    - `_selectedGroup` auf `entity` setzen, um die allgemeinen Entity-Trigger zu fokussieren.

```yaml
  - realm: browser
    listen: show-dialog
    anchor: '&home-assistant $ add-automation-element-dialog'
    rules:
      - '@captured.dialogTag': add-automation-element-dialog
      - "@captured.dialogParams.type": trigger
    directives:
      - type: property
        set: _tab
        value: groups
      - type: property
        set: _selectedGroup
        value: entity
```

## Tools-Schaltfläche zum Sidebar-Titel hinzufügen

Ergebnis: Eine Tools-Schaltfläche, die zu `/config/tools` navigiert.

Vorgehen:

- Auf das Ereignis `uix-broker-ready` im `browser`-Realm lauschen.
- Einen kompakten absoluten Anker für `ha-sidebar` verwenden.
- Die passende Regel greift nur, wenn die Eigenschaft `user.is_admin` des `hass`-Objekts auf `home-assistant` `true` ist. Alternativ könnte `user.is_owner` verwendet werden, um nur den Owner-Benutzer zu treffen.
- Eine `button`-Direktive verwenden, um die Schaltfläche nach dem Titel zu platzieren. Ein einfaches `style`-Objekt setzt einen Box-Shadow und reduziert die Icon-Größe.

```yaml
  - realm: browser
    listen: uix-broker-ready
    anchor: "&home-assistant $ home-assistant-main $ ha-sidebar"
    rules:
      - anchor: "&home-assistant"
        match: "{.hass.user.is_admin=true}"
    directives:
      - type: button
        anchor: "$ div.menu div.title"
        icon: mdi:hammer
        color: purple
        size: s
        tap_action:
          action: navigate
          navigation_path: /config/tools
        style:
          "--ha-button-box-shadow": rgba(0, 0, 0, 0.1) 0px 4px 12px
          "--ha-icon-button-size": 32px
```

![Broker button directive example](../assets/page-assets/broker/broker-button-directive.png){ width="450" }

## Vorgeschlagene Device-Entities-Card für Section-Views wieder zu entities ändern

Ergebnis:

- Die vorgeschlagene Device-Entities-Card für Section-Views wird zu einer `entities`-Card. Hinweis: Das ist nicht dasselbe wie die Vorschläge für andere Views, die auf der Entity-Domain basieren.

Vorgehen:

- Auf das Ereignis `show-dialog` im `browser`-Realm lauschen.
- Die passende Regel greift nur, wenn das `show-dialog`-Ereignis den `dialogTag` `hui-dialog-suggest-card` hat.
- Die Interaktion ist `reentrant: false`, weil sie selbst `show-dialog` auslöst.
- Der Interaktionsanker ist `&home-assistant`. Da die Direktiven `block` enthalten, ist ein synchron vorhandener Anker erforderlich. Alternativ könnte `anchor: target` als Event-Path-Anker verwendet werden, während `anchor: "&home-assistant"` auf der Event-Direktive gesetzt wird.
- Direktiven:
    - Eine `block`-Direktive stoppt die Propagation des ursprünglichen Ereignisses.
    - Eine `event`-Direktive löst das Ereignis erneut mit verändertem `dialogParams.sectionConfig` aus und setzt `cards` auf eine einzelne `entities`-Card. `sectionConfig.type` und `sectionConfig.title` werden mit der `@captured`-Form aus erfassten Daten kopiert. Um nicht den Rest des Ereignisdatenobjekts Eigenschaft für Eigenschaft kopieren zu müssen, führt `capture_data: deep` einen Deep-Merge von `sectionConfig` aus.

```yaml
  - realm: browser
    listen: show-dialog
    debug: true
    reentrant: false
    anchor: "&home-assistant"
    rules:
      - "@captured.dialogTag": hui-dialog-suggest-card
    directives:
      - type: block
      - type: event
        name: show-dialog
        bubbles: true
        composed: true
        capture_data: deep
        data:
          dialogParams:
            sectionConfig:
              type: "@captured.dialogParams.sectionConfig.type"
              title: "@captured.dialogParams.sectionConfig.title"
              cards:
                - type: entities
                  entities: "@captured.dialogParams.entities"
```

## Light-Schaltfläche zum Home-Dashboard-Menüeintrag in der Sidebar

Ergebnis:

- Ähnlich wie [Tools-Schaltfläche zum Sidebar-Titel hinzufügen](#tools-schaltfläche-zum-sidebar-titel-hinzufügen) fügt dieses Beispiel dem Home-Menüeintrag der Sidebar eine Umschaltfläche für ein Licht hinzu. Damit der aktuelle Zustand des Lichts sichtbar wird, verwendet es zusätzlich eine Hilfsinteraktion vom Server-Realm zum Browser-Realm, sodass die Hauptinteraktion läuft, wenn sich der Entity-Zustand ändert.

Vorgehen (Sidebar-Interaktion):

- Auf die Ereignisse `uix-broker-ready` und `uix-update-sidebar` im `browser`-Realm lauschen. `uix-update-sidebar` ist ein benutzerdefiniertes Ereignis; jeder Name ist möglich, solange er zur Hilfsinteraktion passt.
- Einen kompakten absoluten Anker für `ha-sidebar` verwenden.
- Direktiven:
    - Eine `javascript`-Direktive setzt Objektparameter, die in der `button`-Direktive verwendet werden. `icon` und `color` werden anhand des Entity-Zustands gesetzt.
    - Eine `button`-Direktive platziert die Schaltfläche nach dem Home-Menüeintrag. Ein einfaches `style`-Objekt setzt einen Box-Shadow und reduziert die Icon-Größe. Die Aktion ist auf Umschalten des Lichts gesetzt. Hinweis: Konfiguration und Betrieb dieser Schaltfläche folgen dem [UIX Forge Button spark](../forge/sparks/button.md); die Entity wird hier nur für die Aktion einbezogen.

```yaml
  - realm: browser
    listen:
      - uix-broker-ready
      - uix-update-sidebar # custom event from Server realm interaction helper
    anchor: "&home-assistant $ home-assistant-main $ ha-sidebar"
    directives:
      - type: javascript
        id: button_config
        code: |
          const entity = 'light.bed_light';
          const state = hass.states[entity].state;
          return {
            entity: entity,
            icon: state === 'on' ? 'mdi:lightbulb-on' : 'mdi:lightbulb-off',
            color: state === 'on' ? 'var(--state-active-color)' : 'var(--state-inactive-color)'
          };
      - type: button
        anchor: "$ ha-list-item-button#sidebar-panel-home $ a#item div.content"
        icon: "@button_config.icon"
        color: "@button_config.color"
        entity: "@button_config.entity"
        size: s
        tap_action:
          action: toggle
        style:
          "--ha-button-box-shadow": rgba(0, 0, 0, 0.1) 0px 4px 12px
          "--ha-icon-button-size": 32px
          "--uix-button-margin": 6px
```

Vorgehen (Hilfsinteraktion):

Ergebnis:

- Ein benutzerdefiniertes Browser-Ereignis wird ausgelöst, wenn sich der Entity-Zustand ändert. Dadurch reagiert die obige Beispielinteraktion auf Zustandsänderungen von `light.bed_light`.

Vorgehen:

- Auf das Ereignis `state_changed` im Server-Realm lauschen.
- Den Anker über die kompakte absolute Ankerform auf das `home-assistant`-Element setzen.
- Eine `event`-Direktive löst das benutzerdefinierte Browser-Ereignis `uix-update-sidebar` aus, wenn die geänderte `entity_id` `light.bed_light` ist. Dieses Beispiel würde auch auslösen, wenn die `entity_id` `light.other_light` ist. Das ist enthalten, um die Verwendung von `or:` in `match` zu zeigen; du würdest diese Technik verwenden, wenn du dem obigen Beispiel weitere Button-Direktiven hinzufügst.

```yaml
  - realm: server
    listen: state_changed
    anchor: "&home-assistant"
    directives:
      - type: event
        name: uix-update-sidebar
        rules:
          - type: captured
            path: data.entity_id
            match:
              or:
                - light.bed_light
                - light.other_light
```

![Button directive light toggle example](../assets/page-assets/broker/broker-button-light-directive.gif)
