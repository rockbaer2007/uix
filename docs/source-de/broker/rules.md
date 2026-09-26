---
title: Regeln
description: UIX-Broker-Interaktionen gegen Elemente, erfasste Daten und Browser-Identität abgleichen.
---

# Regeln

Jede Interaktionsregel muss passen, bevor Broker seine Direktiven ausführt. Regeln verwenden standardmäßig den Interaktionsanker, können aber auch einen relativen oder absoluten Ersatzanker angeben.

Für Interaktionen ohne `block` versucht UIX Broker einen fehlenden Regel-Ersatzanker alle 50 ms bis zu zwei Sekunden lang erneut aufzulösen. Das ist nützlich für Oberflächen wie Dialoge, die erst nach dem auslösenden Ereignis gemountet werden.

## Host-Element-Regeln

Kompakte String-Regeln verwenden [UIX-Host-Element-Pfad](../concepts/dom.md#hostelement-path-selection)-Matching gegen den Interaktionsanker oder einen Ersatzanker.

Unterstützt werden Selektoren für `Tag`, `class`, `id`, `attribute` und `property`.

Kompakte Regeln werden gegen den Interaktionsanker geprüft und ermöglichen knappe einzeilige Regeldefinitionen.

Die folgende Regelliste passt, wenn der Interaktionsanker:

- `ha-button.action-button[data-action]` ist;
- eine Objekteigenschaft `config.entity` besitzt, die `light.example` entspricht;
- eine Objekteigenschaft `controller` besitzt, die vorhanden, aber `undefined` ist; und
- die Objekteigenschaft `uixBrokerGuard` nicht besitzt.

```yaml
rules:
  - "ha-button.action-button[data-action]"
  - "{.config.entity=light.example}"
  - "{.controller=undefined}"
  - "{!.uixBrokerGuard}"
```

Nutze die erweiterte Form, wenn eine Regel ein anderes Ankerelement prüfen muss. Die `anchor`-Konfiguration wählt das zu prüfende Element aus, und `match` wendet [UIX-Host-Element-Pfad](../concepts/dom.md#hostelement-path-selection)-Matching auf dieses ausgewählte Element an. Ein Regel-`anchor` ist relativ zum Interaktionsanker; stelle `&` voran, um einen absoluten `select_tree`-Pfad ab dem Dokument-Root zu verwenden. Die erweiterte `select_tree`-Form ist ebenfalls verfügbar und immer absolut zum Dokument-Root.

```yaml
rules:
  # Relativer Regelanker mit Tag-Match
  - anchor: "$ ha-dialog"
    match: "ha-dialog"

  # Kompakter absoluter Regelanker mit Tag-Match
  - anchor: "&home-assistant $$ ha-automation-sidebar"
    match: "ha-automation-sidebar"

  # Lange absolute Regel mit Host-Element-Objekteigenschaft
  - anchor:
      select_tree: "home-assistant $$ ha-automation-sidebar"
    match: "{._yamlMode=false}"
```

!!! tip
    Regelanker verwenden dieselbe [select-tree-Syntax](./interaction-anchors.md#select-tree-anker) wie Direktivenanker und werden erneut versucht, während eine Interaktion ohne `block` läuft.

!!! tip
    Der Host-Element-Objekteigenschaftsvergleich `{.property=undefined}` passt nur, wenn die Eigenschaft existiert und ihr Wert `undefined` ist. `{!.property}` passt nur, wenn die Eigenschaft fehlt.

## Typisierte Regeln

Typisierte Regeln haben einen Schlüssel `type`. Unterstützte Typen sind `browserid`, `user`, `user_is_admin`, `hash`, `search`, `captured` und `panel`.

### Browser-Identität

Die Regel `browserid` gleicht eine Browser-ID von [Browser Mod](https://github.com/thomasloven/hass-browser_mod) ab. Verwende den Schlüssel `id`, `browser_id` oder `value` für die erwartete Browser-Identität.

```yaml
rules:
  - type: browserid
    id: kitchen-tablet
```

### Home-Assistant-Benutzer

Verwende `type: user`, um den angemeldeten Home-Assistant-Benutzer entweder über seinen Anzeigenamen (`hass.user.name`) oder seine stabile Benutzer-ID (`hass.user.id`) abzugleichen. Home-Assistant-Benutzernamen sind im Frontend-Benutzerobjekt nicht verfügbar und werden von dieser Regel nicht unterstützt; verwende einen Anzeigenamen oder eine ID. `match` und `value` verwenden dieselbe Matching-Syntax und dieselben Operatoren wie [Regeln für erfasste Daten](#regeln-für-erfasste-daten), einschließlich Wildcards, regulärer Ausdrücke und boolescher Komposition. Setze entweder `match` oder `value`.

```yaml
rules:
  # Passt auf einen Benutzer namens Darryn oder dessen ID Darryn ist.
  - type: user
    match: Darryn

  # Bevorzuge die stabile ID, wenn sie bekannt ist.
  - type: user
    match: 9f1362c9e0a24d918c66d4fdcf12b001
```

Bei einem positiven Matcher darf entweder Name oder ID passen. Ein negierter Matcher, einschließlich `not` oder `!=`, muss beide Felder ausschließen. Dieses Beispiel passt auf jeden Benutzer außer den Benutzer mit dem Namen `wall-panel` oder dieser ID:

```yaml
rules:
  - type: user
    match:
      not: wall-panel
```

Verwende `type: user_is_admin`, um den Administratorstatus des aktuellen Benutzers abzugleichen. Ohne Matcher bedeutet dies „ist Administrator“; setze `match` oder `value` auf `false` für Nicht-Administratoren. Die Regel unterstützt dieselben erweiterten Matcher-Objekte.

Admin-Benutzer:

    rules:
      - type: user_is_admin

Nicht-Admin-Benutzer, dessen Name oder ID mit wall- beginnt:

    rules:
      - type: user
        match: wall-*
      - type: user_is_admin
        match: false

### Browser-URL-Fragment

Verwende `type: hash`, um das Browser-URL-Fragment abzugleichen. Der Wert ist der Teil nach `#`, daher ist kein `path` erforderlich. `match` und `value` verwenden dieselbe Matching-Syntax und dieselben Operatoren wie [Regeln für erfasste Daten](#regeln-für-erfasste-daten).

```yaml
rules:
  - type: hash
    match: settings
```

Diese Regel verhindert, dass die Direktiven der Interaktion ausgeführt werden, sofern die aktuelle URL nicht mit `#settings` endet.

### Browser-Suchparameter

Verwende `type: search`, um einen benannten URL-Suchparameter abzugleichen. Setze `path` auf den Parameternamen. `match` und `value` verwenden dieselbe Matching-Syntax und dieselben Operatoren wie [Regeln für erfasste Daten](#regeln-für-erfasste-daten).

```yaml
rules:
  - type: search
    path: entity_id
    match: "light.kitchen*"
```

Diese Regel verhindert, dass die Direktiven der Interaktion ausgeführt werden, sofern die URL keinen passenden Parameter `?entity_id=` enthält. Verwende `exists: false`, um zu matchen, wenn der benannte Parameter fehlt.

## Regeln für erfasste Daten

Verwende `type: captured`, um Daten abzugleichen, die vom auslösenden Ereignis gesammelt wurden. `path` ist ein durch Punkte getrennter Optional-Chaining-Pfad relativ zu den erfassten Daten; beginne ihn nicht mit `@captured`. Array-Indizes können Punktnotation (`items.0`) oder Klammern (`items[0]`) verwenden. Verwende zitierte Klammer-Schlüssel, wenn eine Eigenschaft Satzzeichen enthält, zum Beispiel `settings['icon-color']`.

Bei Browser- und Shortcut-Interaktionen beginnen erfasste Daten beim `detail` des DOM-Ereignisses. Bei Server-Interaktionen liegen Home-Assistant-Ereignisdaten unter `data`. Array-Indizes werden unterstützt.

```yaml
rules:
  - type: captured
    path: data.new_state.state
    match:
      operator: ">="
      value: 20
```

Einfache Match-Werte unterstützen exakte Werte, Wildcards, reguläre Ausdrücke und numerische Vergleiche:

```yaml
rules:
  - type: captured
    path: button
    match: "save*"
  - type: captured
    path: room
    match: "/^kitchen/i"
  - type: captured
    path: count
    match: ">= 20"
```

### Erweitertes Matching

Ein Matcher-Objekt unterstützt `operator`, `value` oder `match`, `ignore_case`, `exists` sowie verschachtelte `and`-, `or`- und `not`-Kompositionen.

Unterstützte Operatoren sind `>`, `<`, `=`, `<=`, `>=`, `==`, `!=`, `contains`, `starts_with`, `ends_with` und `is_undefined`.

```yaml
rules:
  - type: captured
    path: button
    match:
      or:
        - "save*"
        - "/^submit$/i"
  - type: captured
    path: count
    match:
      and:
        - "> 0"
        - "<= 10"
  - type: captured
    path: data.value
    match:
      operator: is_undefined
      exists: true
```

`is_undefined` mit `exists: true` unterscheidet eine vorhandene Eigenschaft mit dem Wert `undefined` von einem fehlenden Pfad. Verwende `exists: false`, um explizit einen fehlenden Pfad abzugleichen.

### Kompakte Form für erfasste Daten

Für kompakte Konfigurationen kannst du einen oder mehrere Pfade erfasster Daten direkt in einer Objektregel abbilden. Jeder Eintrag muss passen. Das Präfix `@captured` wird nur in dieser kompakten Form beibehalten.

```yaml
rules:
  - "@captured.user.role": admin
    "@captured.enabled": true
```

## Panel-Regeln

Verwende `type: panel`, um das aktuelle UIX-Panel-Objekt abzugleichen. UIX Broker ruft dieses Objekt asynchron ab. Es enthält dieselben `panel`-Felder, die für [Templates](../using/templates.md) verfügbar sind, zum Beispiel `fullUrlPath`, `panelUrlPath`, `viewUrlPath` und `panelComponentName`.

`path` oder sein Alias `property` ist ein durch Punkte getrennter Optional-Chaining-Pfad relativ zu diesem Panel-Objekt. `match` und `value` verwenden exakt dieselbe Matching-Syntax und dieselben Operatoren wie [Regeln für erfasste Daten](#regeln-für-erfasste-daten), einschließlich Wildcards, regulärer Ausdrücke, numerischer Vergleiche, `exists` und `and`/`or`/`not`-Komposition.

```yaml
rules:
  - type: panel
    path: fullUrlPath
    match: "lovelace/kitchen*"
  - type: panel
    path: fullUrlPath
    match:
      operator: contains
      value: automation/edit
  - type: panel
    path: panelComponentName
    match:
      operator: "="
      value: lovelace
```

!!! warning
    Der Panel-Zustand ist asynchron. Eine Interaktion mit einer Panel-Regel in ihren Interaktions-`rules` kann keine `block`-Direktive nutzen, weil das Blockieren eines Ereignisses im synchronen Call-Stack des Ereignisses abgeschlossen sein muss. UIX Broker überspringt solche Interaktionen und protokolliert eine Warnung. Spätere Direktiven ohne `block` können weiterhin eigene Panel-Regeln haben; diese Regeln bedingen nur die jeweilige Direktive, nachdem das Ereignis blockiert wurde.
