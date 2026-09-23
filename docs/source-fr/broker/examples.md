---
title: Exemples
description: Exemples de courtier UIX
---
# Exemples

## Faire de l'onglet de la carte l'onglet par défaut dans la boîte de dialogue d'ajout de carte de l'interface utilisateur

Résultat :

- Sélectionnez l'onglet de la carte lorsque la boîte de dialogue d'ajout de carte s'ouvre.
- Assurez-vous que le premier module d'extension, Suggéré ou Favoris, est ouvert.
- Réduisez tous les autres extensions.

Méthode :

<!-- markdownlint-configure-file {"MD007": { "indent": 4 }} -->
- Écoutez `show-dialog` dans le domaine `browser`.
- La règle de correspondance est transmise uniquement lorsque le `dialogTag` de l'événement `show-dialog` est `hui-dialog-create-card`.
- Utilisez une ancre d'interaction courte et absolue qui correspond à la boîte de dialogue (`hui-dialog-create-card`).
- Directives :
    - Définissez la propriété `_currTab` de la boîte de dialogue sur `card`. Cette propriété étant réactive, il n’est pas nécessaire de forcer une mise à jour.
    - Définissez la propriété `expanded` du premier extenseur sur `true`, à l’aide d’une ancre relative de forme courte.
    - Définissez la propriété `expanded` des autres extensions sur `false`, à l’aide d’ancres relatives de forme courte.

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
    Enregistrez le YAML en tant que nouveau fichier dans votre répertoire ou sous-répertoire de configuration Home Assistant, puis enregistrez-le à l'aide du flux de configuration des options UIX.

## Barre latérale d'automatisation et mode YAML

### Ouvrez la barre latérale de l'éditeur d'automatisation en mode YAML par défaut

Combinez cela avec l'exemple suivant pour permettre de changer le mode YAML ; à lui seul, cet exemple verrouille la barre latérale d'automatisation pour **toujours** utiliser le mode YAML.

Résultat :

- Définissez la barre latérale d'automatisation en mode YAML.
- Ajoutez un bouton dans l’en-tête de la barre latérale d’automatisation pour basculer en mode YAML.

Méthode :

- Écoutez l'événement `open-sidebar` dans le domaine `browser`.
- Définissez `reentrant: false` pour empêcher toute nouvelle saisie lorsque le basculement du mode YAML déclenche lui-même `open-sidebar`.
- L'ancre d'interaction est `manual-automation-editor`, qui est trouvée grâce à une recherche vers l'extérieur à travers le chemin composé de l'événement et les limites de la racine fantôme.
- Utilisez une règle de sélection de chemin d'accès d'élément hôte abrégé pour continuer uniquement lorsque `uixBlockAutoYamlMode` n'existe **pas** sur l'objet JavaScript de l'ancre d'interaction. Ceci est important lorsqu’il est combiné avec l’exemple suivant.
- Utilisez une directive d'appel pour appeler `_toggleYamlMode()` sur `ha-automation-sidebar`, résolu en recherchant la première racine fantôme de l'ancre d'interaction, `manual-automation-editor`.
- Utilisez une directive de bouton pour placer un bouton avant le menu de la barre latérale à trois points. L'action utilisée dans l'action UIX `event` dans laquelle UIX Broker injecte l'élément d'ancrage afin que `toggle-yaml-mode` bouillonne jusqu'à `manual-automation-editor`, permettant l'exemple suivant, [Autoriser le mode YAML dans l'éditeur d'automatisation](#allow-toggle-yaml-mode-in-automation-editor) pour couvrir à la fois le bouton bascule de stock dans la liste déroulante ainsi que le bouton UIX Broker ajouté.

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

### Autoriser le basculement du mode YAML dans l'éditeur d'automatisation

Utilisez cet exemple avec le précédent et l'exemple suivant (un raccourci clavier pour basculer en mode YAML).

L'élément de menu Basculer le mode YAML dans l'éditeur d'automatisation exécute le code qui déclenche `toggle-yaml-mode`, qui appelle finalement `_toggleYamlMode()` sur `manual-automation-editor`. Sans coordination, cela forcerait toujours le mode YAML. Cet exemple contourne ce problème en bloquant l'événement, en définissant une propriété guard, en appelant directement la fonction, puis en effaçant la garde.

Résultat :

- Autorisez l’élément de menu Basculer le mode YAML à basculer entre l’éditeur visuel et le mode YAML.

Méthode :

- Écoutez `toggle-yaml-mode` dans le domaine `browser`.
- L'ancre d'interaction est `manual-automation-editor`, qui est trouvée grâce à une recherche vers l'extérieur à travers le chemin composé de l'événement et les limites de la racine fantôme.
- Directives :
    - Bloquez l'événement car il sera traité directement.
    - Définissez `uixBlockAutoYamlMode` sur `true` sur `manual-automation-editor`.
    - Utilisez une directive d'appel pour appeler `_toggleYamlMode()` sur `ha-automation-sidebar`, résolu en recherchant la première racine fantôme de l'ancre d'interaction, `manual-automation-editor`. Comme l'exemple précédent vérifie l'absence de `uixBlockAutoYamlMode`, il ne poursuit pas sa directive pour forcer le mode YAML.
    - Effacez `uixBlockAutoYamlMode` pour que l'exemple précédent force à nouveau le mode YAML lorsque la barre latérale s'ouvre.

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

### Basculer le mode YAML dans l'éditeur d'automatisation avec un raccourci clavier

Résultat :

- Utilisez un raccourci clavier pour basculer en mode YAML dans l'éditeur d'automatisation.

Méthode :

- Écoutez un raccourci clavier dans le domaine `shortcut` (`$mod+Shift+Y` dans le code ci-dessous ; modifiez-le en conséquence).
- Utilisez l'ancre d'interaction absolue `&home-assistant $$ manual-automation-editor` car la cible du raccourci clavier peut être n'importe quel élément DOM.
- Directives :
    - Définissez `uixBlockAutoYamlMode` sur `true` sur `manual-automation-editor`.
    - Utilisez une directive d'appel pour appeler `_toggleYamlMode()` sur `ha-automation-sidebar`, résolu en recherchant la première racine fantôme de l'ancre d'interaction, `manual-automation-editor`. Comme l'exemple du mode YAML automatique vérifie l'absence de `uixBlockAutoYamlMode`, il ne poursuit pas sa directive pour forcer le mode YAML.
    - Effacez `uixBlockAutoYamlMode` pour que l'exemple de mode YAML automatique force à nouveau le mode YAML lorsque la barre latérale s'ouvre.

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

## Barre latérale d'automatisation et mode YAML terminés

??? example "Complétez YAML pour les trois exemples de barre latérale d'automatisation"
    Enregistrez le YAML en tant que nouveau fichier dans votre répertoire ou sous-répertoire de configuration Home Assistant, puis enregistrez-le à l'aide du flux de configuration des options UIX.
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

## Prioriser les déclencheurs d'entité lors de l'ajout d'un élément d'éditeur d'automatisation

Résultat :

- Accédez directement aux déclencheurs d'entité lors de l'ajout d'un élément d'éditeur d'automatisation.

Méthode :

- Écoutez l'événement `show-dialog` dans le domaine `browser`.
- La première règle de correspondance est transmise uniquement lorsque le `dialogTag` de l'événement `show-dialog` est `add-automation-element-dialog`.
- Les secondes règles de correspondance ne sont transmises que lorsque le type est `trigger` et non d'autres types pouvant être `action` ou `condition`.
- Utilisez une ancre d'interaction courte et absolue qui correspond à la boîte de dialogue (`add-automation-element-dialog`).
- Directives :
    - Définissez la propriété `_tab` sur `groups` ; `groups` est la valeur pour Par type.
    - Définissez `_selectedGroup` sur `entity` pour vous concentrer sur les déclencheurs d'entité générique.

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

## Ajouter un bouton d'outils au titre de la barre latérale

Résultat : un bouton Outils qui ouvre `/config/tools`.

Méthode :

- Écoutez l'événement `uix-broker-ready` dans le domaine `browser`.
- Utilise un ancrage absolu compact pour `ha-sidebar`
- La règle de correspondance n'est transmise que lorsque la propriété `user.is_admin` de l'objet `hass` sur `home-assistant` est vraie (cela pourrait également être `user.is_owner` pour correspondre uniquement à l'utilisateur propriétaire).
- Utilise la directive `button` pour placer le bouton après le titre en utilisant un objet de style simple pour donner une ombre de boîte et une taille d'icône réduite.

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

![Exemple de directive de bouton de courtier](../assets/page-assets/broker/broker-button-directive.png){ width="450" }

## Remplacer la carte suggérée par les entités de périphérique en entités pour les vues en coupe

Résultat :

- Transformez la carte suggérée par les entités de périphérique pour les vues en coupe en une carte d'entités. REMARQUE : Ce n'est pas la même chose que ce qui est suggéré pour d'autres vues basées sur le domaine d'entité.

Méthode :

- Écoutez l'événement `show-dialog` dans le domaine `browser`.
- La règle de correspondance est transmise uniquement lorsque le `dialogTag` de l'événement `show-dialog` est `hui-dialog-suggest-card`.
- L'interaction est définie sur `reentrant: false` car elle déclenche elle-même `show-dialog`.
- L’ancre d’interaction est `&home-assistant`. Étant donné que les directives incluent `block`, une ancre existant de manière synchrone est requise. Alternativement, `anchor: target` pourrait être utilisé comme ancre du chemin d'événement, avec `anchor: "&home-assistant"` défini sur la directive d'événement.
- Directives :
    - Une directive `block` arrête la propagation sur l'événement d'origine.
    - Une directive `event` redistribue l'événement avec un `dialogParams.sectionConfig` modifié, définissant `cards` sur un seul `entities`. `sectionConfig.type` et `sectionConfig.title` sont copiés à partir des données capturées à l'aide du formulaire `@captured`. Pour éviter de copier le reste des données d'événement objet par objet, `capture_data: deep` effectue une fusion approfondie de `sectionConfig`.

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

## Bouton lumineux sur l'élément de menu du tableau de bord Accueil dans la barre latérale

Résultat :

- Semblable à [Ajouter un bouton d'outils au titre de la barre latérale](#add-tools-button-to-sidebar-title), cet exemple ajoute un bouton bascule pour une lumière à l'élément de menu de la barre latérale d'accueil. Pour refléter l'état actuel de la lumière, une interaction d'assistance entre le domaine du serveur et le domaine du navigateur est également utilisée pour que l'interaction principale s'exécute lorsque l'état de l'entité change.

Méthode (interaction dans la barre latérale) :

- Écoutez les événements `uix-broker-ready` et `uix-update-sidebar` dans le domaine `browser`. `uix-update-sidebar` est un événement personnalisé et n'importe quel nom peut être utilisé à condition qu'il corresponde à l'interaction de l'assistant.
- Utilise un ancrage absolu compact pour `ha-sidebar`
- Directives :
    - Directive `javascript` pour définir les paramètres d'objet à utiliser dans la directive `button`. `icon` et `color` sont définis par état d'entité.
    - Utilise la directive `button` pour placer le bouton après l'élément du menu Accueil en utilisant un objet de style simple pour donner une ombre de boîte et une taille d'icône réduite. Action définie pour activer la lumière. REMARQUE : La configuration et le fonctionnement de ce bouton sont conformes au [UIX Forge Button spark](../forge/sparks/button.md), pour lequel l'entité est incluse pour l'action uniquement.

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

Méthode (interaction d'assistance) :

Résultat :

- Déclenchez un événement de navigateur personnalisé lorsque l’état de l’entité change. Cela rend l'exemple d'interaction ci-dessus réactif aux changements d'état pour `light.bed_light`.

Méthode :

- Écoutez l'événement `state_changed` dans le domaine du serveur.
- Placez l'ancre sur l'élément d'assistant à domicile en utilisant une forme d'ancrage compacte absolue.
- Directive `event` pour déclencher l'événement de navigateur personnalisé `uix-update-sidebar` si l'état modifié `entity_id` est `light.bed_light`. Cet exemple se déclencherait également si `entity_id` est `light.other_light`. Ceci est inclus dans cet exemple pour montrer l'utilisation de `or:` dans la correspondance ; vous utiliseriez une telle technique si vous ajoutiez d'autres directives de bouton à l'exemple ci-dessus.

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

![Exemple de basculement de lumière de directive de bouton](../assets/page-assets/broker/broker-button-light-directive.gif)
