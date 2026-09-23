---
title: UIX Broker
description: Configurez les interactions UIX Broker et gérez leurs sources de configuration.
---
# UIX Broker

Une interaction a un `realm`, une valeur `listen`, une interaction `anchor`, un `rules` facultatif et une liste ordonnée de `directives`.

```yaml
uix_broker:
  - realm: shortcut
    debug: true
    listen: "$mod+Shift+Y"
    anchor: '&home-assistant'
    directives:
      - type: action
        action: fire-dom-event
        uix:
          action: toast
          data:
            message: Shortcut pressed
```

Voir [Domaines](./realms.md), [Ancres d'interaction](./interaction-anchors.md), [Règles](./rules.md) et [Directives](./directives.md) pour chaque partie d'une interaction.

## Options d'interaction

| Clé | Description |
| --- | --- |
| `realm` | Où UIX écoute : `browser`, `shortcut` ou `server`. |
| `listen` | Le nom de l'événement DOM, la liaison [Tinykeys](https://jamiebuilds.github.io/tinykeys/) ou le nom de l'événement du bus d'événements Home Assistant pour le domaine sélectionné. Dans le domaine `browser`, il peut également s'agir d'une liste de noms d'événements DOM. |
| `anchor` | L’élément à inspecter et à utiliser comme cible de règle et de directive par défaut. |
| `rules` | Conditions facultatives qui doivent toutes correspondre avant l'exécution des directives. |
| `directives` | Opérations ordonnées à appliquer lorsque l’interaction correspond. |
| `enabled` | La valeur par défaut est `true`. Définissez sur `false` pour conserver une interaction dans la configuration sans l'enregistrer. |
| `reentrant` | La valeur par défaut est `true`. Définissez sur `false` pour ignorer les événements correspondants pour la même interaction pendant sa résolution ou son exécution. |
| `debug` | Définissez sur `true` pour enregistrer le cycle de vie des interactions dans la console du développeur du navigateur. |

Chaque interaction est indépendante. Toutes ses règles doivent correspondre avant l'exécution des directives, et les directives s'exécutent une par une dans l'ordre de configuration.

Utilisez une liste `listen` du domaine du navigateur lorsque la même interaction doit s'exécuter pour plusieurs événements de navigateur :

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

Les listes sont prises en charge uniquement dans le domaine `browser` ; Les interactions `shortcut` et `server` écoutent chacune une liaison ou un nom d'événement.

`reentrant: false` est utile lorsqu'une interaction distribue le même événement qui l'a déclenchée. L'interaction est considérée comme active pendant la résolution des ancres, l'exécution des directives et l'attente des directives.

## Événement prêt pour les courtiers

Une fois qu'UIX Broker a appliqué sa configuration, il distribue un événement de navigateur `uix-broker-ready` sur `window`. L'événement se déclenche une fois que Broker a enregistré ses écouteurs du domaine du navigateur, de sorte qu'une interaction peut écouter cet événement pour appliquer une personnalisation initiale de l'interface utilisateur. Il se déclenche également après chaque rechargement de la configuration du courtier.

Voir [Ajouter un bouton d'outils au titre de la barre latérale](./examples.md#add-tools-button-to-sidebar-title) pour un exemple utilisant cet événement.

## Sources de configuration

Configurez les interactions d'une ou plusieurs des manières suivantes :

1. Dans le flux d'options UIX — **Paramètres → Appareils et services → UIX → Configurer (Cog) →
Configurez le courtier**.
1. Dans un ou plusieurs fichiers YAML enregistrés — **Paramètres → Appareils et services → UIX → Configurer (Cog) →
Gérer les fichiers du courtier**.

Chaque fichier YAML est un mappage avec une liste `uix_broker` de niveau supérieur :

```yaml
uix_broker:
  - realm: browser
    listen: click
    anchor: target
    rules:
      - home-assistant
    directives:
      - type: block
```

Utilisez **Gérer les fichiers Broker** pour enregistrer, désenregistrer ou recharger des fichiers. Les chemins de fichiers peuvent être absolus ou relatifs au répertoire de configuration de Home Assistant. Les fichiers enregistrés sont lus dans l'ordre d'enregistrement, puis les interactions configurées par l'interface utilisateur sont ajoutées. Toutes les interactions sont transmises aux navigateurs connectés sous la forme d'une seule liste.

Les configurations de fichiers YAML utilisent la même résolution YAML de Home Assistant que les fonderies, notamment `!include` et `!secret`. L'action **UIX Broker** dans **Outils → YAML** recharge tous les fichiers Broker enregistrés et les fichiers de rapports.
erreurs. Pour les tableaux de bord en mode YAML, l'action **Actualiser** intégrée du tableau de bord
recharge également les fichiers de courtier enregistrés.

## Chemins d'exécution des interactions synchrones et asynchrones

Les règles de données capturées et d'identité du navigateur s'exécutent de manière synchrone avant la résolution de l'ancre d'interaction. Les ancres d'interaction événement-chemin sont également résolues de manière synchrone. Cela permet à une interaction navigateur-domaine d'appliquer une directive `block` en utilisant les données capturées, l'identité du navigateur et les éléments déjà présents dans le chemin composé de l'événement.

Étant donné que la [directive](directives.md) `block` doit s'exécuter de manière synchrone, les interactions contenant `block` nécessitent que leurs ancres [d'interaction](interaction-anchors.md) et [d'élément hôte](rules.md#host-element-rules) soient immédiatement disponibles. UIX Broker effectue une recherche synchrone ; si l'un ou l'autre n'est pas disponible, il ignore l'interaction.

Une fois qu'une interaction bloquante a été résolue et appliquée `block`, les ancres fournies par les directives ultérieures `property`, `event`, `call` et `button` utilisent toujours le comportement normal de nouvelle tentative asynchrone.

Pour les interactions sans `block`, les [ancres d'interaction](interaction-anchors.md) et les [règles d'élément hôte](rules.md#host-element-rules) manquantes sont réessayées toutes les 50 ms pendant deux secondes maximum. Cela permet à une interaction écoutant un événement de navigateur tel que `show-dialog` d'attendre que la boîte de dialogue soit montée avant de sélectionner la boîte de dialogue ou l'un de ses éléments comme ancre d'interaction.

Voir [Realms](realms.md), [Interaction Anchors](interaction-anchors.md) et [Rules](rules.md)] pour plus d'informations.

## Débogage

Définissez `debug: true` sur une interaction pour enregistrer l'activité de l'auditeur, la résolution de l'ancre, chaque résultat de règle et chaque directive avant et après son exécution. L'entrée post-exécution d'une directive `template` ou `javascript` inclut également son résultat enregistré. Les messages du journal de débogage sont étiquetés avec le domaine et la valeur d'écoute de l'interaction.

```yaml
- realm: browser
  listen: click
  anchor: target
  debug: true
  rules:
    - ".action-button"
  directives:
    - type: event
      name: another-event
```

## Réactivité des interactions

UIX Broker [directives `template` et `javascript`](directives.md) s'exécutent uniquement lorsque leur interaction s'exécute ; ni l’un ni l’autre ne souscrit aux changements d’état. Si vous souhaitez qu'une interaction soit réactive aux mises à jour de l'état de l'entité, vous créez une interaction d'assistance qui écoute `state_changed`, avec une [rule](./rules.md) pour correspondre à l'entité pour laquelle vous souhaitez qu'une interaction soit réactive et utilisez une directive `event` pour déclencher un événement de navigateur personnalisé et ajoutez-le à votre liste d'écoute pour l'interaction.

Interaction entre le domaine du serveur et le domaine du navigateur :

```yaml
  - realm: server
    listen: state_changed
    anchor: "&home-assistant"
    directives:
      - type: event
        name: uix-update-my-interaction
        rules:
          - type: captured
            path: data.entity_id
            match:
              or:
                - switch.bed_light
                - light.bed_light
```

Interaction avec le domaine du navigateur :

```yaml
  - realm: browser
    listen:
      - uix-broker-ready
      - uix-update-my-interaction
    anchor: "&home-assistant $ home-assistant-main $ ha-sidebar"
    #... rules and directives
```

Voir [Bouton lumineux sur l'élément de menu du tableau de bord Accueil sur la barre latérale](./examples.md#light-button-on-home-dashboard-menu-item-on-sidebar) pour un exemple complet.
