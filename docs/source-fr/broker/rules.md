---
title: Règles
description: Faites correspondre les interactions d'UIX Broker avec les éléments, les données capturées et l'identité du navigateur.
---

# Règles

Chaque règle d'interaction doit correspondre avant que Broker exécute ses directives. Les règles utilisent l'ancre d'interaction par défaut, mais peuvent également spécifier une ancre de remplacement relative ou absolue.

Pour les interactions non `block`, UIX Broker réessaye une ancre de remplacement de règle manquante toutes les 50 ms pendant deux secondes maximum. Ceci est utile pour les interfaces, telles que les boîtes de dialogue, qui se montent après le déclenchement de leur événement initiateur.

## Règles des éléments hôtes

Les règles de chaîne compacte utilisent [le chemin d'accès de l'élément hôte UIX](../concepts/dom.md#hostelement-path-selection) correspondant à l'ancre d'interaction ou à une ancre de remplacement.

Les sélecteurs `Tag`, `class`, `id`, `attribute` et `property` sont pris en charge.

Les règles compactes correspondent à l'ancre d'interaction, permettant des définitions de règles concises sur une seule ligne.

La liste de règles ci-dessous correspond au moment où l'interaction s'ancre :

- est `ha-button.action-button[data-action]` ;
- a une propriété d'objet `config.entity` qui est égale à `light.example` ;
- a une propriété d'objet `controller` qui est présente mais `undefined` ; et
- n'a pas la propriété d'objet `uixBrokerGuard`.

```yaml
rules:
  - "ha-button.action-button[data-action]"
  - "{.config.entity=light.example}"
  - "{.controller=undefined}"
  - "{!.uixBrokerGuard}"
```

Utilisez le formulaire développé lorsqu'une règle doit inspecter un élément d'ancrage différent. Sa configuration `anchor` sélectionne l'élément à tester et son `match` applique le [chemin d'accès à l'élément hôte UIX](../concepts/dom.md#hostelement-path-selection) correspondant à cet élément sélectionné. Une règle `anchor` est relative à l'ancre d'interaction ; préfixez-le avec `&` pour un chemin `select_tree` racine absolue du document. Le formulaire `select_tree` étendu est également disponible et est toujours absolu par rapport à la racine du document.

```yaml
rules:
  # Relative rule anchor with a tag match
  - anchor: "$ ha-dialog"
    match: "ha-dialog"

  # Compact absolute rule anchor with a tag match
  - anchor: "&home-assistant $$ ha-automation-sidebar"
    match: "ha-automation-sidebar"

  # Long absolute rule anchor with a host-element object property match
  - anchor:
      select_tree: "home-assistant $$ ha-automation-sidebar"
    match: "{._yamlMode=false}"
```

!!! tip
Les ancres de règle utilisent la même [syntaxe d'arbre de sélection](./interaction-anchors.md#select-tree-anchors) que les ancres de directive et sont réessayées pendant qu'une interaction non-`block` est en cours d'exécution.

!!! tip
Correspondance de propriété d'objet d'élément hôte `{.property=undefined}` correspond uniquement lorsque la propriété existe et que sa valeur est `undefined`. `{!.property}` correspond uniquement lorsque la propriété est absente.

## Règles typées

Les règles typées ont une clé `type`. Les types pris en charge sont `browserid`, `user`, `user_is_admin`, `hash`, `search`, `captured` et `panel`.

### Identité du navigateur

La règle `browserid` correspond à un identifiant de navigateur [Browser Mod](https://github.com/thomasloven/hass-browser_mod). Utilisez la clé `id`, `browser_id` ou `value` pour l'identité de navigateur attendue.

```yaml
rules:
  - type: browserid
    id: kitchen-tablet
```

### Utilisateur de Home Assistant

!!! info
Règles d'utilisation de Home Assistant disponibles dans la version 8.3.0-beta.1

Utilisez `type: user` pour faire correspondre l'utilisateur Home Assistant connecté par son
nom d’affichage (`hass.user.name`) ou identifiant d’utilisateur stable (`hass.user.id`). Maison
Les noms d'utilisateur de l'assistant ne sont pas disponibles dans l'objet utilisateur frontal et ne sont pas
soutenu par cette règle ; utilisez un nom d’affichage ou un identifiant. `match` et `value` utilisent le
mêmes syntaxe et opérateurs de correspondance que [règles de données capturées](#captured-data-rules),
y compris les caractères génériques, les expressions régulières et la composition booléenne. Réglez soit
`match` ou `value`.

```yaml
rules:
  # Matches a user named Darryn or whose id is Darryn.
  - type: user
    match: Darryn

  # Prefer the stable id when it is known.
  - type: user
    match: 9f1362c9e0a24d918c66d4fdcf12b001
```

Pour un correspondant positif, le nom ou l'identifiant peuvent correspondre. Un matcher annulé,
y compris `not` ou `!=`, doit exclure les deux champs. Par exemple, cela correspond
chaque utilisateur sauf l'utilisateur nommé `wall-panel` (ou avec cet identifiant) :

```yaml
rules:
  - type: user
    match:
      not: wall-panel
```

Utilisez `type: user_is_admin` pour correspondre au statut d'administrateur de l'utilisateur actuel.
Sans matcher, cela signifie « est un administrateur » ; réglez `match` ou `value` sur `false` pour
utilisateurs non-administrateurs. Il prend en charge les mêmes objets de correspondance avancés.

Utilisateur administrateur :

    rules:
      - tapez : user_is_admin

Utilisateur non-administrateur dont le nom ou l'identifiant commence par wall- :

    rules:
      - tapez : utilisateur
        match: wall-*
      - tapez : user_is_admin
        match: false

### Fragment d'URL du navigateur

Utilisez `type: hash` pour faire correspondre le fragment d'URL du navigateur. La valeur est la partie après `#`, donc aucun `path` n'est requis. `match` et `value` utilisent la même syntaxe et les mêmes opérateurs de correspondance que les [règles de données capturées](#captured-data-rules).

```yaml
rules:
  - type: hash
    match: settings
```

Cette règle empêche l'exécution des directives de l'interaction à moins que l'URL actuelle ne se termine par `#settings`.

### Paramètres de recherche du navigateur

Utilisez `type: search` pour faire correspondre un paramètre de recherche d'URL nommé. Définissez `path` sur le nom du paramètre. `match` et `value` utilisent la même syntaxe et les mêmes opérateurs de correspondance que les [règles de données capturées](#captured-data-rules).

```yaml
rules:
  - type: search
    path: entity_id
    match: "light.kitchen*"
```

Cette règle empêche l'exécution des directives de l'interaction à moins que l'URL n'ait un paramètre `?entity_id=` correspondant. Utilisez `exists: false` pour effectuer une correspondance lorsque le paramètre nommé est absent.

## Règles relatives aux données capturées

Utilisez `type: captured` pour faire correspondre les données collectées à partir de l'événement initiateur. `path` est un chemin de chaînage facultatif séparé par des points relatif aux données capturées ; ne le démarrez pas avec `@captured`. Les index de tableau peuvent utiliser la notation par points (`items.0`) ou par parenthèses (`items[0]`). Utilisez des touches entre crochets lorsqu'une propriété contient des signes de ponctuation, par exemple `settings['icon-color']`.

Pour les interactions avec le navigateur et les raccourcis, les données capturées commencent au niveau `detail` de l'événement DOM. Pour les interactions avec le serveur, les données d'événement Home Assistant se trouvent sous `data`. Les index de tableau sont pris en charge.

```yaml
rules:
  - type: captured
    path: data.new_state.state
    match:
      operator: ">="
      value: 20
```

Les valeurs de correspondance simples prennent en charge les valeurs exactes, les caractères génériques, les expressions régulières et les comparaisons numériques :

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

### Correspondance avancée

Un objet matcher prend en charge les compositions `operator`, `value` (ou `match`), `ignore_case`, `exists` et les compositions imbriquées `and`, `or` et `not`.

Les opérateurs pris en charge sont `>`, `<`, `=`, `<=`, `>=`, `==`, `!=`, `contains`, `starts_with`, `ends_with` et `is_undefined`.

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

`is_undefined` avec `exists: true` distingue une propriété actuelle dont la valeur est `undefined` d'un chemin manquant. Utilisez `exists: false` pour faire correspondre explicitement un chemin manquant.

### Formulaire compact de données capturées

Pour les configurations compactes, mappez un ou plusieurs chemins capturés directement dans une règle d'objet. Chaque entrée doit correspondre. Le préfixe `@captured` est conservé uniquement sous cette forme compacte.

```yaml
rules:
  - "@captured.user.role": admin
    "@captured.enabled": true
```

## Règles du panel

Utilisez `type: panel` pour faire correspondre l'objet du panneau UIX actuel. UIX Broker obtient cet objet de manière asynchrone ; il contient les mêmes champs `panel` disponibles pour [modèles](../using/templates.md), tels que `fullUrlPath`, `panelUrlPath`, `viewUrlPath` et `panelComponentName`.

`path` (ou son alias `property`) est un chemin de chaînage facultatif séparé par des points par rapport à cet objet panneau. `match` et `value` utilisent exactement la même syntaxe et les mêmes opérateurs de correspondance que les [règles de données capturées](#captured-data-rules), y compris les caractères génériques, les expressions régulières, les comparaisons numériques, `exists` et la composition `and`/`or`/`not`.

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
L’état du panneau est asynchrone. Une interaction utilisant une règle de panneau ne peut pas utiliser une directive `block`, car le blocage d'un événement doit se terminer dans la pile d'appels synchrones de l'événement. UIX Broker ignore ces interactions et enregistre un avertissement.
