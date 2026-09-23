---
title: Délai d'application des styles aux dialogues
description: Découvrez comment UIX peut appliquer les styles d'un dialogue après son ouverture afin de réduire les scintillements et les défauts d'animation.
---
# Délai d'application des styles aux dialogues

Par défaut, UIX applique les styles dès l'ouverture des dialogues. Sur certains appareils, cela peut provoquer un bref scintillement, car le navigateur redessine le dialogue pendant son animation. L'option de **délai d'application des styles** attend la fin de l'animation d'ouverture, ce qui évite ces scintillements et défauts d'animation.

## Activation dans l'interface de l'intégration

Le délai d'application des styles est **désactivé par défaut**. Pour l'activer :

1. Dans Home Assistant, ouvrez **Paramètres → Appareils et services → UI eXtension → Configurer**.
2. Choisissez **Performance settings** dans le menu.
3. Activez **Delay UIX styling for dialogs until fully shown**.
4. Enregistrez.

Le réglage prend effet immédiatement dans toutes les sessions de navigateur connectées ; aucun rechargement de page n'est nécessaire.

## Fonctionnement

Lorsque cette option est activée, UIX attend l'événement `after-show` émis par les dialogues Home Assistant à la fin de leur animation d'ouverture. Les styles sont alors appliqués, plutôt qu'au début de l'ouverture, ce qui évite les recalculs en cours d'animation susceptibles de provoquer un scintillement.

!!! info
    Le besoin de retarder l'application des styles peut dépendre du navigateur. Safari et les appareils WebKit, notamment sous iOS, semblent davantage concernés par les problèmes d'application immédiate des styles.

## API de remplacement côté client

`window.uixCoordinator` expose la méthode `setDialogApplyAfterShowOverride()`. Les intégrations externes, comme [Browser Mod](https://github.com/thomasloven/hass-browser_mod), peuvent ainsi appliquer un réglage **par navigateur**, **par utilisateur** ou **par appareil**, sans modifier la configuration du serveur. Avec Browser Mod, utilisez une action JavaScript [**Default action**](https://github.com/thomasloven/hass-browser_mod/blob/master/documentation/configuration-panel.md#default-action).

Cette valeur de remplacement est prioritaire sur la configuration de l'intégration envoyée par le serveur.

```js
// Enable the delay for this browser session:
window.uixCoordinator.setDialogApplyAfterShowOverride(true);

// Disable the delay for this browser session:
window.uixCoordinator.setDialogApplyAfterShowOverride(false);

// Remove the override and revert to server-configured defaults:
window.uixCoordinator.setDialogApplyAfterShowOverride(null);
```

Comme `ha-dialog` et `ha-more-info-dialog` lisent la valeur du coordinateur à l'ouverture, le remplacement s'applique dès le prochain dialogue, sans rechargement de page.

### Utilisation avec Browser Mod

Browser Mod permet d'exécuter du JavaScript pour chaque session de navigateur avec une action [**Default action**](https://github.com/thomasloven/hass-browser_mod/blob/master/documentation/configuration-panel.md#default-action). Il convient donc aux réglages par appareil. Par exemple, pour activer le délai sur une tablette murale lente :

```yaml
# In your Browser Mod configuration for a specific browser ID:
- action: browser_mod.javascript
  code:
    window.uixCoordinator?.setDialogApplyAfterShowOverride(true);
```

### Utilisation avec `custom:button-card`

Vous pouvez ajouter au tableau de bord un bouton [`custom:button-card`](https://github.com/custom-cards/button-card) pour activer ou désactiver le délai à la volée.

```yaml
# Enable the dialog styling delay for this browser session:
type: custom:button-card
name: Enable Dialog Delay
icon: mdi:timer-play-outline
tap_action:
  action: javascript
  javascript: |
    [[[ window.uixCoordinator?.setDialogApplyAfterShowOverride(true); ]]]

---

# Disable the dialog styling delay for this browser session:
type: custom:button-card
name: Disable Dialog Delay
icon: mdi:timer-off-outline
tap_action:
  action: javascript
  javascript: |
    [[[ window.uixCoordinator?.setDialogApplyAfterShowOverride(false); ]]]
```

??? example "Exemple complet de bouton bascule"
    ```yaml
    type: custom:button-card
    grid_options:
      rows: 2
      columns: 6
    section_mode: true
    update_timer: 500ms
    variables:
      dialogApplyAfterShow: |
        [[[ return window.uixCoordinator?.dialogApplyAfterShow; ]]]
    styles:
      card:
        - "--dialog-icon-color": >
            [[[ return variables.dialogApplyAfterShow ? "var(--state-active-color)"
            : "var(--state-inactive-color)"; ]]]
        - "--ha-ripple-hover-color": >
            [[[ return variables.dialogApplyAfterShow ? "var(--state-active-color)"
            : "var(--state-inactive-color)"; ]]]
    name: >
      [[[ return variables.dialogApplyAfterShow ? "UIX Dialog Delay ON" : "UIX
      Dialog Delay OFF"; ]]]
    icon: >
      [[[ return variables.dialogApplyAfterShow ? "mdi:timer-play-outline" :
      "mdi:timer-off-outline"; ]]]
    color: var(--dialog-icon-color)
    tap_action:
      action: javascript
      javascript: >
        [[[ variables.dialogApplyAfterShow ?
        window.uixCoordinator?.setDialogApplyAfterShowOverride(false) :
        window.uixCoordinator?.setDialogApplyAfterShowOverride(true); ]]]
    ```

## Référence de configuration

| Réglage | Valeur par défaut | Description |
|---|---|---|
| Delay UIX styling for dialogs until fully shown | Désactivé | Lorsque cette option est activée, UIX applique les styles du dialogue après la fin de l'animation d'ouverture. |

## Quand utiliser ce délai

Ce délai est utile lorsque :

- les dialogues présentent un **bref scintillement ou des défauts d'animation**, notamment avec la variante « bottom sheet » ;
- vous utilisez un **appareil lent ou peu puissant** et les calculs de styles pendant l'ouverture sont perceptibles.

!!! warning
    Lorsque cette option est activée, les styles sont appliqués un peu plus tard. La différence est généralement faible sur les appareils rapides, mais peut devenir perceptible sur les appareils très lents. Si seuls certains appareils en ont besoin, utilisez Browser Mod et une action **Default action** pour appliquer le réglage en JavaScript.
