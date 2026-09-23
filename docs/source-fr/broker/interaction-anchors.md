---
title: Ancres d’interaction
description: Sélectionnez l'élément que les règles et directives UIX Broker utilisent par défaut.
---
# Ancres d’interaction

Une ancre d'interaction sélectionne l'élément que les règles d'élément hôte inspectent et que les directives utilisent par défaut. Les interactions de navigateur et de raccourci peuvent effectuer une sélection à partir du chemin composé de l'événement initiateur ou utiliser un chemin UIX `select_tree`. Les interactions serveur utilisent uniquement les chemins `select_tree`.

Pour un YAML concis, la configuration `anchor` est généralement écrite sous forme compacte. Le tableau ci-dessous résume quand la sélection du chemin d'événement est utilisée et quand `select_tree` est utilisé.

| `anchor:` | Méthode |
| --- | --- |
| `target` | Utilise la sélection [event-path](#event-path-anchors) et se résout vers la cible d'un événement `browser` ou `shortcut` [realm](realms.md). Il n'est pas disponible dans le domaine `server`. |
| `<`, `<$`, `<selector> <$` ou `<selector> <$$` | Utilise la sélection [event-path](#event-path-anchors). Il n'est pas disponible dans le domaine `server`. |
| Une chaîne commençant par `&` | Utilise la sélection `select_tree` de `document`. Il est disponible dans tous les [realms](realms.md). |
| `{ select_tree: <path> }` | Utilise la sélection longue `select_tree` de `document`. Il est disponible dans tous les domaines. |

## Ancres de chemin d'événement

Les expressions de chemin d'événement sont évaluées de droite à gauche à partir du `target` implicite, où `target` est l'élément le plus interne renvoyé par [`event.composedPath()`](https://developer.mozilla.org/en-US/docs/Web/API/Event/composedPath).

| Ancre | Résultat |
| --- | --- |
| `target` | La cible d’origine de l’événement la plus interne. Dans toutes les autres formes, `target` est facultatif par souci de concision mais peut être inclus pour des raisons de lisibilité. |
| `< [target]` | L'élément parent de `target`. |
| `<$ [target]` | Le premier hôte racine fantôme au-dessus de `target`. |
| `<selector> <$ [target]` | Le premier élément correspondant dans le DOM clair de ce premier hôte fantôme. |
| `<selector> <$$ [target]` | Le premier élément correspondant en marchant vers l'extérieur à travers le chemin composé et en traversant les racines de l'ombre. |

!!! note
Les ancres d'interaction événement-chemin sont résolues de manière synchrone et peuvent être utilisées avec la [directive `block`](directives.md#block).

```yaml
# Nearest shadow host of the event target
anchor: "<$"

# A matching ha-automation-row element in the target host's light DOM
anchor: "ha-automation-row <$"

# Le premier élément ha-automation-row correspondant dans le chemin composé vers l'extérieur, y compris les limites de Shadow Root
anchor: "ha-automation-row <$$"
```

`<` et `<$` acceptent explicitement `target` à droite et peuvent être inclus pour des raisons de lisibilité, mais il est plus compact de l'omettre. `<$$` nécessite un sélecteur extérieur.

## Ancres d'arbre de sélection

Utilisez un chemin UIX `select_tree` normal lorsqu'une ancre n'est pas déterminée par le chemin de l'événement, y compris chaque interaction du serveur.

!!! note
Les chemins `select_tree` sont décrits en détail dans [DOM navigation](../concepts/dom.md).

```yaml
# Compact absolute form
anchor: "&home-assistant $ hui-dialog-create-card"

# Long form, always absolute from document without needing `&`
anchor:
  select_tree: "home-assistant $ home-assistant-main $ ha-panel-lovelace $ hui-root"
```

Pour les interactions non-`block`, UIX Broker réessaye une ancre `select_tree` manquante toutes les 50 ms pendant deux secondes maximum. Ceci est utile pour les interfaces, telles que les boîtes de dialogue, qui se montent après le déclenchement de leur événement initiateur.

## Ancres dans les règles et directives

Les règles et directives utilisent l'ancre d'interaction par défaut. Ils peuvent également remplacer l'ancre d'interaction par leur propre configuration `anchor`, sous forme relative ou absolue compacte, ou sous la forme longue `select_tree`. Pour la forme absolue compacte, le chemin commence par `&`.

Relative:

```yaml
- type: property
  # property directive anchor is relative to the interaction anchor (a dialog in this example)
  anchor: "$ ha-dialog div.body hui-card-picker $ div#content>ha-expansion-panel:nth-of-type(1)"
  set: expanded
  value: false
```

Forme abrégée absolue :

```yaml
- type: call
  anchor: "&home-assistant $ ha-more-info-dialog"
  method: closeDialog
```

Forme longue absolue :

```yaml
- type: call
  anchor:
    select_tree: "home-assistant $ ha-more-info-dialog"
  method: closeDialog
```

## Rechercher des chemins dans la console du navigateur

Pour trouver un chemin d'ancrage absolu, sélectionnez un élément dans l'inspecteur du navigateur et exécutez :

```javascript
uix_broker_absolute_path($0)
```

Cela signale un chemin d’ancrage d’interaction absolu compact commençant par `&`.

Pour une ancre de directive ou de règle relative à une interaction précédemment résolue
ancre, utilisez :

```javascript
uix_broker_path($0)
```

Il choisit l’ancre d’interaction récente la plus proche. Passer l'ancre
explicitement comme deuxième argument lorsque plusieurs interactions se chevauchent :

```javascript
uix_broker_path($0, $1)
```

Voir [Aide à l'inspection DOM](../concepts/dom.md#uix_broker_path0-broker-directive-anchor-helper)
pour les détails de l'aide à la console.
