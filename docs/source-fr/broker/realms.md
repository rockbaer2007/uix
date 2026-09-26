---
title: Royaumes
description: Choisissez où une interaction UIX Broker écoute les événements.
---
# Royaumes

L'interaction `realm` détermine où Broker écoute et comment sa valeur `listen` est interprétée.

| Royaume | `listen` | Source de l'événement | Support d'ancrage |
| --- | --- | --- | --- |
| `browser` | Nom de l'événement DOM, tel que `click` ou `show-dialog` | Événements du navigateur sur `window` pendant la phase de capture | Expressions de chemin d'événement à partir du chemin composé et des chemins `select_tree` |
| `shortcut` | [Liaison de touches Tinykeys](https://jamiebuilds.github.io/tinykeys/), telle que `"$mod+Shift+K"` | Événement clavier du navigateur | Expressions de chemin d'événement à partir du chemin composé et des chemins `select_tree` |
| `server` | Bus d'événements Home Assistant [nom event](https://www.home-assistant.io/docs/configuration/events/) tel que `state_changed`, `component_loaded` ou `call_service` | Connexion frontale active | Chemins `select_tree` uniquement |

Tous les domaines prennent en charge les [règles](rules.md) et les [directives](directives.md). L'élément [interaction Anchor](interaction-anchors.md) sélectionné est toujours dans le navigateur actuel, donc un événement capturé à partir du domaine `server` peut toujours mettre à jour un élément du navigateur ou lui envoyer un événement.

## Enregistrements des écouteurs

Broker enregistre un écouteur navigateur pour chaque nom d'événement distinct du domaine `browser` et un abonnement au bus d'événements Home Assistant pour chaque nom d'événement distinct du domaine `server`. Les interactions activées qui utilisent la même valeur `listen` partagent cet enregistrement ; Broker évalue leurs ancres et leurs règles après la réception de l'événement.

Par exemple, deux interactions `browser` qui écoutent toutes deux `uix-applied` utilisent un seul écouteur `window`, et deux interactions `server` qui écoutent toutes deux `state_changed` utilisent un seul abonnement au bus d'événements. Vous pouvez séparer les interactions selon leur cible, leurs règles ou leurs directives lorsque cela rend la configuration plus claire : les regrouper n'est pas nécessaire pour réduire le nombre d'écouteurs.

## Navigateur

`browser` écoute `window` pendant la phase de capture. Utilisez-le pour les événements DOM tels que `click`, `change`, `show-dialog` et les événements de navigateur personnalisés de Home Assistant. `listen` peut être un nom d'événement ou une liste lorsque la même interaction doit répondre à plusieurs événements.

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

Par exemple, exécutez une interaction après le démarrage de Broker et chaque fois qu'une mise à jour du panneau émet `uix-update` :

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

L'objet `detail` de l'événement de navigateur est la racine des données capturées. Voir [Règles de données capturées](./rules.md#captured-data-rules) et [Directive d'événement](./directives.md#event) pour savoir comment les données capturées sont mises en correspondance et
réutilisé.

### Événements du cycle de vie de UIX Styling

UIX Styling émet les événements navigateur suivants, qui remontent dans le DOM et traversent les frontières des composants, depuis son élément `<uix-node>` :

- `uix-applied` — après l'ajout ou la réapplication de UIX à un élément. L'événement peut se déclencher à nouveau lorsque l'élément hôte est mis à jour ou que la configuration UIX est réappliquée ; les directives doivent donc pouvoir être exécutées plusieurs fois sans effet indésirable.
- `uix-styles-update` — lorsque le nœud UIX met à jour le texte CSS rendu, y compris après une mise à jour pilotée par un modèle. Le dernier texte est disponible dans `detail.uix_node._rendered_styles`, mais Lit n'a pas encore appliqué son élément `<style>`. Pour lire les styles calculés dans une directive ultérieure, utilisez d'abord une directive [`action: javascript`](./directives.md#action-javascript) avec `data.code: "return event.detail.uix_node.updateComplete;"`. L'action attend cette Promise avant l'exécution de la directive suivante.
- `uix-theme-update` — après le nouveau traitement d'une mise à jour du thème par le nœud UIX. Cet événement est émis même si le CSS UIX obtenu est inchangé.

Les trois événements fournissent l'élément `<uix-node>` d'origine dans `detail.uix_node`. Utilisez l'ancre de chemin d'événement `"< target"` pour sélectionner son élément parent, auquel UIX est appliqué, que le nœud se trouve dans le DOM léger ou dans un DOM fantôme.

Par exemple, définissez `themeMode` d'une carte de carte géographique lorsque UIX est appliqué à la carte ou lorsque son thème est mis à jour :

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

## Raccourci

`shortcut` utilise [Tinykeys](https://jamiebuilds.github.io/tinykeys/) pour enregistrer une combinaison de touches de navigateur sur `window`. Sa valeur `listen` utilise la syntaxe Tinykeys. `$mod` signifie `Meta` sur macOS et `Control`
sous Windows et Linux.

```yaml
- realm: shortcut
  listen: "$mod+Shift+Y"
  anchor: target
  directives:
    - type: call
      method: focus
```

Les raccourcis clavier peuvent utiliser une touche, un code, des modificateurs et des séquences. Home Assistant utilise également Tinykeys pour ses propres raccourcis. Utilisez des raccourcis clavier qui n’entrent pas en conflit avec les raccourcis de Home Assistant, du navigateur ou du système d’exploitation. Vous pouvez désactiver les raccourcis clavier de Home Assistant pour le navigateur afin de rendre ces raccourcis clavier disponibles pour UIX Broker, qui continue d'enregistrer les raccourcis clavier lorsque les raccourcis clavier de Home Assistant sont désactivés.

Le `KeyboardEvent` initiateur est disponible pour les actions JavaScript sous le nom `event`. Son chemin composé peut également être utilisé par [ancres d'interaction](./interaction-anchors.md).

## Serveur

`server` s'abonne au bus d'événements Home Assistant via la connexion frontale active. Il n'a pas de cible d'événement de navigateur, son ancre d'interaction doit donc utiliser les chemins `select_tree`.

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

Pour les interactions avec le serveur, les données capturées possèdent une clé `data` contenant la charge utile de l'événement Home Assistant (par exemple, `data.entity_id` ou `data.new_state.state`). Dans les règles et directives, les données capturées peuvent être référencées avec `"@captured.data..."` ; les guillemets sont requis pour `@` dans YAML.

## Blocage

La directive `block` est disponible dans les domaines du navigateur et des raccourcis. Ses règles d'ancrage et d'élément hôte sont résolues de manière synchrone ; si une ancre `select_tree` ne peut pas être trouvée immédiatement, l'interaction complète est ignorée. Cela préserve la propagation du navigateur et le timing des actions par défaut. Les événements du serveur ne peuvent pas être bloqués.

!!! note
    Tinykeys ignore les touches enfoncées dans les zones de saisie, de zone de texte, de sélection et modifiables, donc un domaine `shortcut` `block` ne s'exécutera pas dans ces situations. Pour bloquer une clé dans ces situations, utilisez le domaine `browser` avec `listen: keydown`, ainsi que des règles de données capturées et/ou d'éléments hôtes synchrones.

    Lorsqu'une liaison de raccourci s'exécute, `block` empêche l'action native par défaut et arrête les écouteurs ultérieurs sur `window`. Il ne peut pas annuler un gestionnaire de raccourcis Home Assistant déjà exécuté.

## Modèles

UIX Broker ne fournit délibérément pas de domaine qui s'abonne directement aux modèles Jinja2. La directive [`template`](./directives.md#template) peut restituer un modèle une fois pendant qu'une interaction est en cours, mais elle n'écoute pas les modifications ultérieures. Pour un comportement réactif, utilisez un script, une automatisation ou une entité modèle avec un déclencheur, puis déclenchez un événement personnalisé sur le bus d'événements Home Assistant et écoutez-le dans le domaine `server`.

!!! tip
    Vous pouvez utiliser l'intégration [`custom_event`](https://github.com/reubn/hass_custom_event) pour déclencher des événements personnalisés sur le bus d'événements Home Assistant, puis les écouter dans le domaine `server`.
