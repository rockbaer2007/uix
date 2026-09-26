---
title: Directives
description: "Appliquer des opérations déclaratives UIX Broker à un élément sélectionné."
---
# Directives

Les directives s'exécutent une par une après chaque correspondance de règle d'interaction. Chaque directive effectue une opération configurée, en utilisant l'ancre d'interaction par défaut ou une ancre de directive explicitement sélectionnée si elle est prise en charge. À l'exception de `block`, une directive peut également avoir son propre `rules` ; la directive ne s'exécute que lorsque toutes correspondent, sinon Broker l'ignore et passe à la directive suivante.

- [Block](#block) — empêche l'action et la propagation par défaut de l'événement initiateur du navigateur.
- [Property](#property) — définit ou efface une propriété d'objet JavaScript.
- [Event](#event) — envoie un `CustomEvent`.
- [Call](#call) — invoque une méthode d'élément.
- [Button](#button) — insérez un bouton interactif Home Assistant.
- [Badge](#badge) — insérez un badge d'état stylisé avec Web Awesome.
- [Contenu textuel](#text-content) — insérez du texte stylisé à côté d'un élément.
- [Icône de vignette](#tile-icon) — insérez une icône de vignette interactive Home Assistant.
- [Info-bulle](#tooltip) — associez une info-bulle stylisée à un élément.
- [Verrou](#lock) — exigez un défi de déverrouillage avant d'utiliser un élément.
- [Gestionnaire d'actions](#action-handler) — associez des actions Home Assistant à un élément existant.
- [Action](#action) — exécutez une action Home Assistant, frontend ou UIX.
- [Template](#template) — affiche un modèle Jinja2 une fois et enregistre son résultat.
- [JavaScript](#javascript) — évalue JavaScript de manière synchrone et enregistre sa valeur de retour.
- [Wait](#wait) — retarde la prochaine directive.

## Règles de la directive

Ajoutez `rules` à n’importe quelle directive à l’exception de `block` pour conditionner uniquement cette directive. La syntaxe est la même que celle des [règles d'interaction](./rules.md). Pour `property`, `event`, `call`, `action-handler`, `button`, `badge`, `text-content`, `tile-icon`, `tooltip` et `lock`, les règles d'élément hôte inspectent par défaut l'ancre de directive résolue. Pour `action`, `template`, `javascript` et `wait`, elles inspectent l’ancre d'interaction. Une directive `event` ciblant `window` ou `document` utilise également l'ancre d'interaction. L'`anchor` d'une règle reste relatif à cette ancre par défaut, ou peut être absolu comme d'habitude.

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

Les règles `panel` obtiennent l'état actuel du panneau lors de sa première utilisation, puis le réutilisent pour le reste de l'interaction. Si l'interaction possède elle-même une règle `panel`, les règles des directives réutilisent cet état. Une règle `panel` au niveau d'une directive permet à une directive antérieure de s'exécuter quel que soit le panneau actuel, tandis qu'une directive ultérieure ne s'exécute que si le panneau correspond.

`block` n'accepte pas les règles de directive. Placez sa condition dans le `rules` de l'interaction afin que l'événement soit bloqué de manière synchrone uniquement lorsque l'interaction complète correspond.

## Bloquer

`block` appelle `preventDefault()` et `stopImmediatePropagation()` sur l'événement de navigateur initiateur.

```yaml
- type: block
```

Il est disponible uniquement dans les domaines `browser` et `shortcut`. L'ancre d'interaction et les ancres de règle d'élément hôte doivent être résolues de manière synchrone ; si une ancre `select_tree` requise n'est pas déjà présente, UIX Broker ignore l'interaction complète. Une directive `block` est appliquée avant que les directives restantes ne soient traitées, même lorsqu'elle apparaît plus tard dans la liste.

## Ancres de directive

Les directives `property`, `event`, `call`, `button` et `tile-icon` utilisent l'ancre d'interaction par défaut. Chacun peut remplacer cette valeur par défaut avec sa propre configuration `anchor`. Une chaîne nue est relative à l'ancre d'interaction, une chaîne commençant par `&` est un chemin `select_tree` de racine de document absolu compact et `{ select_tree: ... }` est la forme absolue longue équivalente.

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

Utilisez l'assistant de console `uix_broker_path($0)` dans la console du navigateur pour rechercher un chemin d'ancrage de directive relatif.

Voir [Interaction Anchors](./interaction-anchors.md#anchors-in-rules-and-directives) pour les formats de sélection.

Voir [Recherche de chemins dans la console du navigateur](./interaction-anchors.md#finding-paths-in-the-browser-console) pour plus d'informations sur les aides de console disponibles.

## Propriété

La directive `property` modifie l'objet JavaScript de l'ancre sélectionnée. `set` prend un chemin de propriété séparé par des points, crée tous les niveaux d'objet simple intermédiaires manquants et attribue la valeur à la propriété finale. `clear` emprunte le même type de chemin et supprime uniquement la propriété finale ; il ne supprime pas ses objets parents.

```yaml
- type: property
  set: config.heading
  value: New title
- type: property
  clear: config.icon
```

Les valeurs peuvent faire référence à des données capturées ou à un résultat `template` ou `javascript` précédent. `@captured` résout l'objet de données capturées complet, tandis que `@captured.path` résout la valeur sur ce chemin séparé par des points. Les index de tableau peuvent utiliser la notation par points (`items.0`) ou par parenthèses (`items[0]`) ; utilisez une clé entre crochets entre guillemets pour les propriétés d'objet contenant des signes de ponctuation, telles que `settings['icon-color']`. La référence est remplacée avant que la propriété ne soit définie et doit être citée en YAML car elle commence par `@`.

```yaml
- type: property
  set: config.entity
  value: "@captured.entity_id"
```

Les directives `template` et `javascript` sauvegardent leur valeur sous leur `id`. Une directive ultérieure peut utiliser `@id` ou une propriété telle que `@id.path` ; la valeur conserve son type d'origine, y compris les objets et les tableaux. Les références occupent une valeur YAML complète — Broker ne les interpole pas dans une chaîne plus longue.

## Événement

`event` envoie un `CustomEvent`. Son `target` est par défaut `anchor`, ce qui signifie l'ancre de directive sélectionnée (ou l'ancre d'interaction lorsqu'aucune ancre de directive n'est définie). Définissez `target: window` ou `target: document` pour qu'il soit distribué globalement ; ces cibles n'utilisent ni ne résolvent d'ancre de directive spécifique à un événement. `bubbles` et `composed` sont par défaut `false`, correspondant à l'API DOM.

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

Définissez `capture_data: true` pour copier les données d'événement capturées dans un événement modifié. Le `detail` de l'événement sortant commence par les données capturées de l'interaction initiatrice, puis superpose superficiellement les valeurs de l'objet `data` de cette directive. L'option `capture_data` est disponible uniquement pour la directive `event`.

```yaml
- type: event
  name: broker-forwarded-event
  capture_data: true
  data:
    source: uixBroker
```

Définissez `capture_data: deep` lorsque les objets simples imbriqués doivent être fusionnés à la place. La directive `data` l'emporte pour les valeurs contradictoires ; les tableaux et les objets non simples sont remplacés en tant que valeurs complètes. Cela laisse `capture_data: true` inchangé.

```yaml
- type: event
  name: broker-forwarded-event
  capture_data: deep
  data:
    params:
      source: uixBroker
```

## Appeler

`call` invoque une méthode sur l'ancre sélectionnée. `method` accepte un chemin de méthode sécurisé séparé par des points et préserve la liaison `this` de l'objet méthode. `args`, lorsqu'il est fourni, doit être un tableau et prend en charge la substitution des données capturées.

```yaml
- type: call
  method: focus
- type: call
  method: setSelectionRange
  args: [0, 5]
```

Bouton ##

`button` insère un Home Assistant `ha-button` à côté de l'ancre de directive. Il utilise la même configuration de bouton et la même gestion des actions que le [bouton Forge spark](../forge/sparks/button.md). Le bouton est inséré par défaut après l'ancre de la directive.

Utilisez `after` ou `before` pour sélectionner un autre élément de référence. Ces chemins sont relatifs à l'ancre de directive résolue et prennent en charge la syntaxe UIX `select_tree` habituelle. Le bouton est toujours inséré en tant que frère de l’élément de référence correspondant.

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

Utilisez `style` pour un mappage plat des noms et valeurs des propriétés CSS. Les propriétés sont définies en ligne sur le `ha-button` généré, ce qui est utile pour les dimensions et l'espacement des boutons qui ne peuvent pas être stylisés à partir de la configuration du tableau de bord.

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

Utilisez `uix` pour le style UIX, y compris les styles à l’intérieur de la racine fantôme du bouton. Son type UIX est `uix-broker-button` et les paramètres de bouton résolus sont disponibles sous la forme `config` dans les modèles UIX.

!!! info
    Style UIX `button` disponible dans la version 8.3.0-beta.3

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

| Clé | Tapez | Par défaut | Descriptif |
| --- | --- | --- | --- |
| `after` | `string` | ancre directive | Sélecteur relatif pour l'élément de référence. Le bouton est inséré après. |
| `before` | `string` | — | Sélecteur relatif pour l'élément de référence. Le bouton est inséré avant lui. |
| `entity` | `string` | — | ID d'entité utilisé par les actions basées sur l'entité. |
| `icon` | `string` | — | Icône MDI placée dans la fente pour étiquette du bouton. Il est prioritaire sur `label`. |
| `color` | `string` | — | Couleur de l’icône pour un bouton contenant uniquement une icône. |
| `label` | `string` | `""` | Etiquette du bouton. |
| `start_icon` / `end_icon` | `string` | — | Icône MDI avant ou après l'étiquette. |
| `variant` | `string` | Par défaut de Home Assistant | `brand`, `neutral`, `danger`, `warning` ou `success`. Les boutons contenant uniquement des icônes sont par défaut `neutral`. |
| `appearance` | `string` | Par défaut de Home Assistant | `accent`, `filled`, `outlined` ou `plain`. Les boutons contenant uniquement des icônes sont par défaut `plain`. |
| `size` | `string` | — | `s` (petit) ou `m` (moyen). |
| `style` | objet | — | Carte plate des noms de propriétés CSS et des valeurs de chaîne ou numériques, définie en ligne sur `ha-button`. |
| `uix` | objet | — | Configuration UIX appliquée au bouton généré en tant que type `uix-broker-button`. |
| `tap_action` / `hold_action` / `double_tap_action` | actions | — | Action Home Assistant à exécuter à partir du bouton. |

!!! note
    - Définissez au maximum un des `after` et `before`.
    - Les clics sur les boutons sont isolés du propre gestionnaire d'action de l'élément de référence.
    - Les événements de pointeur, de souris, de toucher et de clic s'arrêtent au bouton généré. Cela empêche l'ondulation ou le gestionnaire d'action d'un élément conteneur de réagir tout en conservant l'action et l'ondulation du bouton.
    - La même variable CSS `--uix-button-margin` que l'étincelle du bouton Forge s'applique. La marge par défaut est `-6px` pour un bouton étiqueté et `0px` pour un bouton contenant uniquement une icône.
    - D'autres variables CSS applicables à l'étincelle du bouton Forge s'appliquent également.

## Icône de tuile

!!! info
    Directive `tile-icon` disponible en 8.3.0-beta.3


`tile-icon` insère un Home Assistant `ha-tile-icon` à côté de l'ancre directive. Il utilise le même rendu d'icône et la même gestion des actions que [Forge Tile-icon spark](../forge/sparks/tile-icon.md). L'icône de vignette est insérée par défaut après l'ancre de la directive.

Utilisez `after` ou `before` pour sélectionner un autre élément de référence. Ces chemins sont relatifs à l'ancre de directive résolue et prennent en charge la syntaxe UIX `select_tree` habituelle. L'icône de tuile est insérée en tant que frère de l'élément de référence correspondant.

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

Utilisez `style` pour un mappage plat des noms et valeurs des propriétés CSS. Les propriétés sont définies en ligne sur le `ha-tile-icon` généré, ce qui est utile pour positionner et dimensionner l'icône là où le style du tableau de bord ne peut pas l'atteindre.

```yaml
- type: tile-icon
  entity: light.living_room
  style:
    margin-inline-start: 8px
    "--tile-icon-size": 28px
    z-index: 1
```

Utilisez `uix` pour le style UIX, y compris les styles à l’intérieur de la racine fantôme de l’icône de vignette. Son type UIX est `broker-tile-icon`, et les paramètres d'icône de tuile résolus sont disponibles en tant que `config` dans les modèles UIX.

```yaml
- type: tile-icon
  entity: light.living_room
  uix:
    style: |
      :host {
        --tile-icon-size: {{ '32px' if is_state(config.entity, 'on') else '24px' }};
      }
```

| Clé | Tapez | Par défaut | Descriptif |
| --- | --- | --- | --- |
| `after` | `string` | ancre directive | Sélecteur relatif pour l'élément de référence. L'icône de tuile est insérée après. |
| `before` | `string` | — | Sélecteur relatif pour l'élément de référence. L'icône de tuile est insérée devant elle. |
| `entity` | `string` | — | Entité dont l'icône d'état est rendue. Il fournit l'action d'appui par défaut : `toggle` pour les entités basculables, sinon `none`. |
| `icon` | `string` | — | Icône MDI. Avec `entity`, il remplace l'icône d'état normal de l'entité. |
| `icon_path` | `string` | — | Chemin SVG transmis à `ha-tile-icon` sous le nom `iconPath`. |
| `image_url` | `string` | — | URL de l'image transmise à `ha-tile-icon` sous le nom `imageUrl`. |
| `color` | Couleur CSS | — | Couleur de l’icône de tuile. Avec `entity`, ceci est appliqué pendant que l'entité est active. |
| `style` | objet | — | Carte plate des noms de propriétés CSS et des valeurs de chaîne ou numériques, définie en ligne sur `ha-tile-icon`. |
| `uix` | objet | — | Configuration UIX appliquée à l'icône de vignette générée en tant que type `broker-tile-icon`. |
| `tap_action` / `hold_action` / `double_tap_action` | actions | — | Action Home Assistant à exécuter à partir de l’icône de la vignette. |

!!! note
    - Définissez au plus un des `after` et `before`.
    - Fournissez une source d'icônes avec `icon`, `icon_path`, `image_url` ou `entity`.
    - Les icônes de vignettes basées sur les entités sont mises à jour lorsque l'état de Home Assistant est mis à jour.
    - Les événements de pointeur, de souris, de toucher et de clic s'arrêtent à l'icône générée. Cela empêche l'ondulation ou le gestionnaire d'action d'un élément conteneur de réagir tout en conservant l'action et l'ondulation de l'icône de tuile.
    - Broker ajoute l'attribut `data-uix-broker-tile-icon` à chaque icône de tuile générée, afin qu'il puisse être sélectionné à partir du style UIX.

## Action

`action` exécute un appel de service Home Assistant, une action frontale standard ou l'une des actions spécifiques à UIX Broker.

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

### Action JavaScript

`action: javascript` est une action de courtier UIX. Mettez le code dans `data.code`. UIX Broker transmet automatiquement `hass`, `anchor`, `event` et `captured` en tant que variables. `hass` est l'objet Home Assistant actif, `anchor` est l'élément DOM d'ancrage d'interaction résolu, `event` est l'événement initiateur et `captured` est les données capturées de l'interaction.

```yaml
- type: action
  action: javascript
  data:
    code: |
      console.log(anchor, event, captured)
```

Utilisez JavaScript uniquement à partir de configurations UIX fiables.

## Info-bulle

`tooltip` ajoute une info-bulle Home Assistant `wa-tooltip` à côté de la cible sélectionnée. Ses options et variables CSS correspondent à la [spark d'info-bulle Forge](../forge/sparks/tooltip.md). Par défaut, `for` désigne l'ancre de directive résolue ; un sélecteur est relatif à cette ancre et utilise la syntaxe UIX habituelle de `select_tree`. La cible doit être un élément, et non une racine terminale de DOM fantôme.

```yaml
- type: tooltip
  content: Ouvrir les commandes de l'éclairage du salon
  placement: bottom
```

Utilisez `for: previous` pour associer l'info-bulle au dernier élément créé par une directive précédente de la même interaction. Cette option fonctionne avec `button`, `badge`, `text-content`, `tile-icon` et `lock` (qui crée une superposition de verrouillage). Les directives qui ne créent pas d'élément ne modifient pas cette référence.

```yaml
- type: button
  icon: mdi:lightbulb
  tap_action:
    action: toggle
- type: tooltip
  for: previous
  content: Allumer ou éteindre l'éclairage
  placement: bottom
```

```yaml
- type: tooltip
  for: "$ ha-dialog ha-icon-button"
  content: Fermer
  without_arrow: true
```

Utilisez `style` pour définir une table simple de propriétés CSS, notamment les variables `--uix-tooltip-*` directement sur l'info-bulle générée.

```yaml
- type: tooltip
  for: previous
  content: Allumer ou éteindre l'éclairage
  style:
    "--uix-tooltip-background-color": var(--primary-color)
    "--uix-tooltip-content-color": white
    "--uix-tooltip-max-width": 24ch
```

`trigger` accepte les modes d'activation Web Awesome séparés par des espaces : `hover`, `focus`, `click` et `manual`. Avec `hover`, l'info-bulle reste ouverte lorsque le pointeur passe de la cible à son contenu, ce qui permet de faire défiler les contenus contraints. `manual` ne déclenche pas l'ouverture automatiquement ; utilisez `open` pour définir son état lors de l'exécution de la directive.

```yaml
- type: tooltip
  for: previous
  trigger: manual
  open: true
  content: Cette info-bulle est ouverte par la directive
```

| Clé | Type | Valeur par défaut | Description |
| --- | --- | --- | --- |
| `for` | chaîne | ancre de directive | Sélecteur de cible, ou `previous` pour l'élément créé par la directive précédente. |
| `content` | chaîne ou nombre | `""` | Contenu HTML de l'info-bulle. Les nombres sont affichés comme du texte. |
| `placement` | chaîne | `"top"` | `top`, `top-start`, `top-end`, `bottom`, `bottom-start`, `bottom-end`, `left`, `left-start`, `left-end`, `right`, `right-start` ou `right-end`. |
| `distance` | nombre | `8` | Écart en pixels entre l'info-bulle et la cible. |
| `skidding` | nombre | `0` | Décalage en pixels le long de l'axe de la cible. |
| `show_delay` | nombre | `150` | Délai en millisecondes avant l'affichage. |
| `hide_delay` | nombre | `150` | Délai en millisecondes avant le masquage. |
| `trigger` | chaîne | `"hover focus"` | Modes d'activation séparés par des espaces : `hover`, `focus`, `click` ou `manual`. |
| `open` | booléen | `false` | Définit l'état d'ouverture lors de l'exécution de la directive, notamment avec `trigger: manual`. |
| `without_arrow` | booléen | `false` | Masque la flèche directionnelle. |
| `style` | objet | — | Dictionnaire simple de propriétés CSS et de valeurs, appliquées en ligne à `wa-tooltip`. |

L'info-bulle est insérée comme élément frère de sa cible. Définissez les variables CSS `--uix-tooltip-*` sur le parent de la cible ou sur un ancêtre pour la personnaliser ; consultez la [référence des variables CSS de la spark d'info-bulle Forge](../forge/sparks/tooltip.md#css-variables-reference).

## Modèle

`template` restitue un modèle Home Assistant Jinja2 une fois via l'API du modèle ; il ne crée pas d'abonnement modèle. Son résultat de chaîne est stocké sous `id` pour les directives restantes de cette interaction.

Chaque rendu non mis en cache est un aller-retour vers le serveur Home Assistant. Évitez de l'utiliser sur des interactions qui peuvent s'exécuter fréquemment. Définissez `cache` sur un nombre positif de millisecondes lorsqu'une valeur légèrement obsolète est acceptable :

```yaml
- type: template
  id: example
  cache: 5000
  template: "{{ states('sensor.example') }}"
```

Le cache est conservé dans le navigateur et partagé par les directives de modèle en utilisant le même texte de modèle et les mêmes résultats de directive précédente. Une valeur mise en cache est utilisée uniquement lorsqu'elle est inférieure à la durée `cache` de la directive ; `cache: 0` (ou en omettant `cache`) est toujours restitué. Le cache stocke uniquement les résultats réussis, est effacé lors du rechargement de la configuration du Broker et n'observe pas les modifications de modèle pendant la période de cache. Lorsque `cache` est activé, les résultats des directives précédentes doivent être sérialisables en JSON car ils font partie de la clé de cache ; les objets circulaires ne peuvent pas être mis en cache.

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

`id` doit commencer par une lettre ou un trait de soulignement et peut ensuite contenir des lettres, des chiffres, des traits de soulignement et des traits d'union. Le nom `captured` est réservé aux données d'événement `@captured` et ne peut pas être utilisé comme identifiant. Utilisez des chemins de tableau de points ou de crochets pour sélectionner un objet ou une valeur de tableau enregistré, tout comme pour `@captured`. Les clés de support citées fonctionnent également, par exemple `@config_path['icon-color']` ou `@config_path["icon-color"]`.

Les modèles reçoivent les résultats des directives antérieures dans la variable `directive` de niveau supérieur. Par exemple, une directive antérieure avec `id: provider` est disponible sous le nom `{{ directive.provider }}`. Cet espace de noms contient uniquement les résultats des directives précédentes dans la même interaction.

## JavaScript

`javascript` évalue une fois `code` et enregistre sa valeur de retour synchrone sous `id`. Le code reçoit `hass`, `anchor`, `event`, `captured` et `directive` ; `directive` contient des résultats de directives antérieures issus de la même interaction. Renvoyez un scalaire, un objet ou un tableau ; les directives suivantes peuvent l'utiliser comme `@id` sans conversion. Les mêmes [exigences concernant les identifiants](#template) que pour la directive `template` s'appliquent. Une Promise renvoyée n'est pas attendue ; utilisez une directive [`action: javascript`](#action-javascript) qui renvoie une Promise si les directives suivantes doivent attendre la fin du travail asynchrone.

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

Utilisez JavaScript uniquement à partir de configurations UIX fiables.

## Attendez

Utilisez `wait` pour suspendre une séquence de directives sans effectuer une autre opération. Cela nécessite un nombre de millisecondes non négatif.

```yaml
directives:
  - type: wait
    wait: 500
  - type: action
    action: light.turn_on
    target:
      entity_id: light.example
```

Chaque directive accepte également `wait`, un nombre non négatif de millisecondes. Sous cette forme, UIX Broker attend après avoir appliqué la directive avant de démarrer la suivante. Une directive `block` s'exécute toujours de manière synchrone, bien qu'elle puisse inclure `wait` pour retarder les directives ultérieures.

```yaml
directives:
  - type: event
    name: broker-started-event
    wait: 250
  - type: action
    action: light.turn_on
```
