---
title: Actions UIX
description: Découvrez les actions UIX : vider le cache, afficher des informations détaillées ou une notification, protéger une action par un code, exécuter du JavaScript et envoyer des événements du navigateur.
---
# Actions UIX

UIX propose plusieurs actions personnalisées utilisables dans les tableaux de bord Home Assistant. Elles sont appelées avec `action: fire-dom-event` ; l'objet `uix:` indique l'action et l'objet `data:` contient ses paramètres. Vous pouvez les utiliser avec toute carte prenant en charge `fire-dom-event`, notamment les cartes standard de Home Assistant.

!!! info
    Les paramètres de l'objet `data:` d'une action UIX dépendent de l'événement Home Assistant appelé et ne sont pas définis par UIX. La configuration comporte donc plusieurs clés `action`, ce qui peut prêter à confusion. Chacune a toutefois son rôle. En cas de problème, suivez la configuration indiquée pour l'action UIX concernée.

```yaml
# ... card config
  tap_action:
    action: fire-dom-event
    uix:
      action: <action>
      data:
        <action-data>
```

!!! info
    `action: clear_cache` et `action: more_info` sont également acceptés et correspondent respectivement à `action: clear-cache` et `action: more-info`.

## `clear-cache` — vider le cache du frontend Home Assistant

Cette action vide le cache de l'application frontend Home Assistant et recharge le navigateur, sans toucher à `localStorage`. Elle est pratique lorsque l'option est cachée dans un menu de débogage. Elle vide également d'autres données du cache de l'application ; `localStorage`, qui contient notamment l'identifiant du navigateur Browser Mod, reste intact.

| Configuration | Paramètre | Valeur par défaut | Description |
| --- | --- | --- | --- |
| `action: clear-cache` | — | — | Vide le cache de l'application Home Assistant et recharge le navigateur. |
| `data:` | — | — | Non utilisé |

Exemple de bouton pour vider le cache et recharger la page :

```yaml
show_name: true
show_icon: true
type: button
name: Clear Frontend Cache
tap_action:
  action: fire-dom-event
  uix:
    action: clear-cache
```

## `event` — envoyer un événement du navigateur

Envoie un [`CustomEvent`](https://developer.mozilla.org/en-US/docs/Web/API/CustomEvent) sur `window`. Cela permet de relier l'action d'un bouton à une interaction UIX Broker dans le realm `browser`, sans ajouter d'écouteur `fire-dom-event` chargé de déballer un autre événement.

| Configuration | Paramètre | Valeur par défaut | Description |
| --- | --- | --- | --- |
| `action: event` | — | — | Envoie un `CustomEvent` du navigateur sur `window`. |
| `name` | **OBLIGATOIRE** | — | Nom de l'événement. |
| `data` | — | `{}` | Propriété `detail` de l'événement. |

Pour une action de tableau de bord classique, l'événement est envoyé sur `window`. Pour une directive `button` d'UIX Broker, UIX l'envoie automatiquement depuis le point d'insertion du bouton — sa cible `after`, sa cible `before` ou l'ancre de la directive — avec `bubbles: true` et `composed: true`. L'interaction Broker qui le reçoit peut donc utiliser une ancre de chemin d'événement telle que `target`, `<` ou `<$`, sans rechercher à nouveau un chemin absolu `select_tree`.

Par exemple, ce bouton Broker envoie l'événement `toggle-yaml-mode` depuis son point d'insertion :

```yaml
- type: button
  before: $ ha-automation-sidebar $$ ha-automation-sidebar-card $ ha-dialog-header slot:nth-of-type(3) ha-dropdown
  icon: mdi:code-braces
  tap_action:
    action: fire-dom-event
    uix:
      action: event
      name: toggle-yaml-mode
      data:
        source: sidebar-button
```

## `more-info` — afficher les informations détaillées d'une entité

Affiche la boîte de dialogue Home Assistant des informations détaillées. Vous pouvez choisir la vue affichée à son ouverture.

| Configuration | Paramètre | Valeur par défaut | Description |
| --- | --- | --- | --- |
| `action: more-info` | — | — | Affiche la boîte de dialogue des informations détaillées avec les options d'entité et de vue définies dans `data:`. |
| `data:` | — | — | Options d'entité et de vue pour les informations détaillées. |
| | `entity` | — | Identifiant de l'entité à afficher. |
| | `view` | `info` | Vue initiale de la boîte de dialogue : `info`, `history`, `settings`, `related`, `add_to` ou `details`. |

Exemple d'affichage des informations détaillées avec la vue historique :

```yaml
type: tile
entity: light.bed_light
tap_action:
  action: fire-dom-event
  uix:
    action: more-info
    data:
      entity: light.bed_light
      view: history
```

## `toast` — afficher une notification Home Assistant

Affiche une notification temporaire (toast) de Home Assistant.

| Configuration | Paramètre | Valeur par défaut | Description |
| --- | --- | --- | --- |
| `action: toast` | — | — | Affiche une notification temporaire Home Assistant avec les options de `data:`. |
| `data:` | — | — | Options de la notification. |
| | `id` | — | Identifiant de la notification. Réutilisez le même identifiant pour remplacer une notification existante. |
| | `message` | **OBLIGATOIRE** | Chaîne ou objet. Une chaîne n'est pas traduite. Pour utiliser une traduction Home Assistant, indiquez un objet avec `translationKey` et, éventuellement, `args`. |
| | `duration` | `4000` | Durée d'affichage en millisecondes. Toute valeur inférieure à 4 000 est ramenée à 4 000 (4 secondes). Utilisez `-1` pour un affichage permanent ; une nouvelle notification la remplacera. |
| | `dismissable` | `false` | Affiche une icône de fermeture qui permet à l'utilisateur de masquer immédiatement la notification. |
| | `bottomOffset` | `0` | Décalage vertical positif par rapport au bas de la fenêtre du navigateur. Les notifications apparaissent à `--ha-space-4` (16 px par défaut) au-dessus de la limite de la zone sûre. |
| | `action` | — | Si elle est définie, affiche un bouton qui exécute `action.tap_action` au clic. |
| | `action.primary` | — | Si la valeur est `true`, affiche le bouton `action` avec le style principal rempli. La variante du bouton est toujours `brand`. |
| | `action.text` | **OBLIGATOIRE** | Chaîne ou objet définissant le texte du bouton. Une chaîne n'est pas traduite ; pour utiliser une traduction Home Assistant, indiquez un objet avec `translationKey` et, éventuellement, `args`. |
| | `action.tap_action` | **OBLIGATOIRE** | Configuration d'action Home Assistant. |
| | `secondary_action` | — | Si elle est définie, affiche à gauche du bouton `action` un bouton qui exécute `secondary_action.tap_action`. |
| | `secondary_action.primary` | — | Si la valeur est `true`, affiche le bouton `secondary_action` avec le style principal rempli. La variante du bouton est toujours `brand`. |
| | `secondary_action.text` | **OBLIGATOIRE** | Chaîne ou objet définissant le texte du bouton. Une chaîne n'est pas traduite ; pour utiliser une traduction Home Assistant, indiquez un objet avec `translationKey` et, éventuellement, `args`. |
| | `secondary_action.tap_action` | **OBLIGATOIRE** | Configuration d'action Home Assistant. |

Exemple de notification avec une action dont le libellé est traduit :

```yaml
type: tile
entity: light.bed_light
tap_action:
  action: fire-dom-event
  uix:
    action: toast
    data:
      message: "Bed Light"
      duration: 10000
      dismissable: true
      bottomOffset: 550
      action:
        primary: true
        text:
          translationKey: ui.dialogs.more_info_control.light.toggle
        tap_action:
          action: perform-action
          perform_action: light.toggle
          target:
            entity_id: light.bed_light
      secondary_action:
        text: Custom
        tap_action:
          action: perform-action
          perform_action: light.toggle
          target:
            entity_id: light.ceiling_lights
```

![Exemple d'action de notification UIX](../assets/page-assets/extras/extra-toast-action.gif)

## `javascript` — exécuter du JavaScript dans la session du navigateur

Exécute du code JavaScript dans la session du navigateur. L'objet `hass` est disponible, ainsi qu'un objet facultatif `variables`.

!!! warning
    Cette action exécute du JavaScript arbitraire dans la session frontend Home Assistant actuelle. N'utilisez que du code et une configuration de confiance ; ce code peut accéder aux données disponibles dans la session du navigateur.

| Configuration | Paramètre | Valeur par défaut | Description |
| --- | --- | --- | --- |
| `action: javascript` | — | — | Exécute du JavaScript avec les options définies dans `data:`. |
| `data:` | — | — | Options JavaScript. |
| | `code` | **OBLIGATOIRE** | Code JavaScript à exécuter. |
| | `variables` | `{}` | Objet de variables facultatif. Chaque variable nommée est accessible en JavaScript avec `variables.<name>`. Les valeurs peuvent être de n'importe quel type. |

Exemple d'action JavaScript utilisant une variable et l'objet `hass` pour éteindre une lumière :

```yaml
type: tile
entity: light.bed_light
tap_action:
  action: fire-dom-event
  uix:
    action: javascript
    data:
      variables:
        entity_id: light.bed_light
      code: |
        console.log("UIX: Custom javascript action executed!");
        hass.callService("light", "turn_off", {}, { entity_id: variables.entity_id });
```

## `locked_action` — demander un code ou une confirmation avant l'action

!!! info
    Disponible à partir de la version 8.3.0-beta.2.

Exécute une action Home Assistant classique uniquement après validation du verrou configuré pour l'utilisateur actuel. Cette action convient notamment au redémarrage, à l'ouverture d'un portail ou à la modification d'un réglage important, sans devoir envelopper toute la carte dans un verrou Forge.

```yaml
type: button
name: Restart Home Assistant
tap_action:
  action: fire-dom-event
  uix:
    action: locked_action
    data:
      locks:
        - code: 1234
          admins: true
      locked_action:
        action: perform-action
        perform_action: homeassistant.restart
```

!!! warning
    `locked_action` protège une interaction dans le frontend ; il ne constitue pas une limite d'autorisation. Toute personne capable de modifier le tableau de bord ou d'inspecter sa configuration chargée peut voir le code et l'action protégée. Pour contrôler les accès, utilisez les permissions Home Assistant et des contrôles côté serveur.

### Référence de configuration

| Clé | Type | Valeur par défaut | Description |
| --- | --- | --- | --- |
| `locked_action` | object | — | **Obligatoire.** Action Home Assistant à exécuter après validation du verrou. |
| `locks` | list | `[]` | Liste ordonnée de règles de verrouillage. Consultez [Correspondance des verrous](#lock-matching) et [Clés d'une règle](#lock-entry-keys). |
| `permissive` | boolean | `false` | Si la valeur est `true`, les utilisateurs ne correspondant à aucune règle peuvent exécuter l'action. |
| `entity` | string | — | Identifiant d'entité transmis à l'action imbriquée, pour les types d'action qui utilisent l'entité de la carte. |
| `code_dialog` | object | — | Libellés de la demande de code ou de phrase secrète. Consultez [Boîte de dialogue du code](#code-dialog). |
| `id` | string ou number | — | Identifiant stable utilisé pour suivre les nouvelles tentatives et les blocages. Fortement recommandé avec `retry_delay` ou `max_retries` ; utilisez un identifiant différent pour chaque action protégée. |

`locked_action` accepte tout objet d'action Home Assistant classique, notamment `perform-action`, `toggle`, `more-info`, `navigate` et `fire-dom-event`.

<a id="lock-matching"></a>
### Correspondance des verrous

`locks` est une liste ordonnée. La première règle active correspondante détermine le contrôle demandé. Si aucune règle active ne correspond, la première règle correspondante avec `active: false` autorise l'action sans contrôle.

| Configuration | Utilisateurs concernés |
| --- | --- |
| La liste `users` est présente | Utilisateurs dont le nom figure dans la liste. Avec `admins: true`, tous les administrateurs correspondent aussi. |
| Aucune liste `users` | Tous les utilisateurs non administrateurs, sauf ceux de `except`. |
| Aucune liste `users` avec `admins: true` | Tous les utilisateurs, sauf ceux de `except`. |

`admins` ajoute les administrateurs à la règle : sans cette option, ils en sont exclus sauf s'ils figurent dans `users`. Si aucune règle ne correspond, `permissive: true` autorise l'action. Avec la valeur par défaut `permissive: false`, les administrateurs contournent le verrou d'interaction et les autres utilisateurs ne peuvent pas exécuter l'action.

<a id="lock-entry-keys"></a>
### Clés d'une règle de verrouillage

| Clé | Type | Valeur par défaut | Description |
| --- | --- | --- | --- |
| `active` | boolean | `true` | Définissez `false` pour autoriser explicitement les utilisateurs correspondants sans contrôle. |
| `code` | string ou number | — | Code à saisir. Un code uniquement numérique affiche le pavé numérique Home Assistant ; les autres valeurs utilisent un champ de mot de passe. |
| `pin` | string ou number | — | Alias de `code`. |
| `confirmation` | string, boolean ou object | — | Confirmation demandée après le code. `true` utilise le texte par défaut de Home Assistant ; une chaîne définit un texte personnalisé ; un objet peut contenir `title` et `text`. |
| `users` | liste de chaînes | — | Noms des utilisateurs concernés par cette règle. |
| `admins` | boolean | `false` | Étend la règle aux administrateurs. Sans liste `users`, elle s'applique alors à tous les utilisateurs. |
| `except` | liste de chaînes | — | Noms des utilisateurs exemptés d'une règle sans `users`. |
| `retry_delay` | number ou string | — | Délai après un code erroné avant une nouvelle tentative. Les nombres sont en millisecondes ; les chaînes acceptent des unités comme `"10s"`. |
| `max_retries` | number | — | Nombre de codes erronés autorisés avant le blocage prolongé. |
| `max_retries_delay` | number ou string | `30000` | Durée du blocage après `max_retries`. Les nombres sont en millisecondes ; les chaînes acceptent des unités comme `"30s"` ou `"5m"`. |

<a id="code-dialog"></a>
### Boîte de dialogue du code

Utilisez `code_dialog` pour personnaliser la demande de code ou de phrase secrète.

| Clé | Type | Valeur par défaut | Description |
| --- | --- | --- | --- |
| `title` | string | Valeur par défaut Home Assistant | Titre de la boîte de dialogue. |
| `submit_text` | string | Valeur par défaut Home Assistant | Libellé du bouton de confirmation. |
| `cancel_text` | string | Valeur par défaut Home Assistant | Libellé du bouton d'annulation. |

### Suivi des tentatives

L'état des tentatives est conservé dans la session de navigateur actuelle. Définissez `id` dès que le délai entre les tentatives ou le blocage est important, en particulier si un modèle ou une autre carte personnalisée peut recréer la configuration du tableau de bord : un identifiant explicite reste stable après la recréation du contrôle. Le même identifiant partage le compteur de tentatives ; attribuez donc un identifiant distinct à chaque action protégée.

```yaml
tap_action:
  action: fire-dom-event
  uix:
    action: locked_action
    data:
      id: restart-home-assistant
      code_dialog:
        title: Enter administrator PIN
        submit_text: Restart
      locks:
        - code: 1234
          admins: true
          max_retries: 3
          max_retries_delay: 5m
      locked_action:
        action: perform-action
        perform_action: homeassistant.restart
```
