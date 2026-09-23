---
title: Limiter les mises à jour d'état du frontend
description: Découvrez comment UIX limite la fréquence des mises à jour d'état de Home Assistant afin de fluidifier le frontend.
---
# Limiter les mises à jour d'état du frontend

Home Assistant peut envoyer très fréquemment des états au frontend, sans permettre de choisir ceux qui sont transmis. Par exemple, une valeur RSSI Bluetooth qui change rapidement provoque l'envoi de nouveaux états et l'actualisation des vues. Cela peut gêner les appareils lents ou les tableaux de bord très chargés. UIX permet de limiter la fréquence des mises à jour d'état afin d'atténuer ces problèmes.

!!! info "Quelles mises à jour sont limitées ?"
    Seules les mises à jour qui modifient `hass.states` sont limitées. Tous les autres changements de `hass` — comme les thèmes, la langue ou l'utilisateur connecté — sont transmis immédiatement afin de préserver la réactivité de l'interface.

## Activation dans l'interface de l'intégration

La limitation des mises à jour d'état est **désactivée par défaut**. Pour l'activer :

1. Dans Home Assistant, ouvrez **Paramètres → Appareils et services → UI eXtension → Configurer**.
2. Choisissez **Performance settings** dans le menu.
3. Activez **Throttle entity state updates**.
4. Réglez **Throttle interval**, c'est-à-dire le délai minimal en millisecondes entre deux rendus déclenchés par un changement d'état. La valeur par défaut est `200 ms` ; les valeurs autorisées vont de 50 à 10 000 ms.
5. Enregistrez.

Le réglage prend effet immédiatement dans toutes les sessions de navigateur connectées ; aucun rechargement de page n'est nécessaire.

## Fonctionnement

Lorsque la limitation est activée, UIX adapte `hui-view` — l'élément qui enveloppe chaque tableau de bord Home Assistant — en lui ajoutant une vérification `shouldUpdate`. Si les états changent plus souvent que l'intervalle configuré, les mises à jour intermédiaires sont temporairement ignorées.

### Appliquer la dernière mise à jour

Une mise à jour ignorée pendant la limitation n'est **jamais perdue**. UIX programme un minuteur après chaque mise à jour limitée. Il se déclenche `intervalle de limitation + 50 ms` après la **dernière** mise à jour ignorée, puis applique celle-ci à la vue. L'interface finit ainsi toujours par afficher le dernier état connu.

Si une mise à jour non limitée est transmise avant le déclenchement du minuteur, celui-ci est annulé et aucun rendu supplémentaire n'est effectué.

## API de remplacement côté client

`window.uixCoordinator` expose la méthode `setThrottleOverride()`. Les intégrations externes, comme [Browser Mod](https://github.com/thomasloven/hass-browser_mod), peuvent ainsi appliquer une limite **par navigateur**, **par utilisateur** ou **par appareil**, sans modifier la configuration du serveur. Avec Browser Mod, utilisez une action JavaScript [**Default action**](https://github.com/thomasloven/hass-browser_mod/blob/master/documentation/configuration-panel.md#default-action).

Cette valeur de remplacement est prioritaire sur la configuration de l'intégration envoyée par le serveur. Chaque champ peut être remplacé séparément ; tout champ omis conserve la valeur définie par le serveur.

```js
// Enable throttle with a 500 ms interval for this browser session:
window.uixCoordinator.setThrottleOverride({ enable: true, ms: 500 });

// Override only the interval (inherits the server enable/disable flag):
window.uixCoordinator.setThrottleOverride({ ms: 1000 });

// Remove the override and revert to server-configured defaults:
window.uixCoordinator.setThrottleOverride(null);
```

Comme `hui-view` lit les valeurs du coordinateur à chaque appel de `shouldUpdate`, le remplacement prend effet immédiatement, sans rechargement de page.

### Utilisation avec Browser Mod

Browser Mod permet d'exécuter du JavaScript pour chaque session de navigateur avec une action [**Default action**](https://github.com/thomasloven/hass-browser_mod/blob/master/documentation/configuration-panel.md#default-action). Il convient donc aux réglages par appareil. Par exemple, pour appliquer un intervalle plus long sur une tablette murale lente :

```yaml
# In your Browser Mod configuration for a specific browser ID:
- action: browser_mod.javascript
  code:
    window.uixCoordinator?.setThrottleOverride({ enable: true, ms: 1000 });
```

### Utilisation avec `custom:button-card`

Vous pouvez ajouter au tableau de bord des boutons [`custom:button-card`](https://github.com/custom-cards/button-card) pour activer, désactiver ou effacer le remplacement à la volée, ce qui est utile pour les tests ou pour les tableaux de bord nécessitant une commande dynamique.

```yaml
# Enable throttling at 500 ms for this browser session:
type: custom:button-card
name: Enable Throttle
icon: mdi:speedometer
tap_action:
  action: javascript
  javascript: |
    [[[ window.uixCoordinator?.setThrottleOverride({ enable: true, ms: 500 }); ]]]

---

# Disable throttling (clear override) for this browser session:
type: custom:button-card
name: Disable Throttle
icon: mdi:speedometer-slow
tap_action:
  action: javascript
  javascript: |
    [[[ window.uixCoordinator?.setThrottleOverride(null); ]]]
```

??? example "Exemple complet de bouton bascule"
    Cette carte bouton lit l'état actuel de la limitation, l'active ou la désactive et affiche l'intervalle lorsqu'elle est active.
    ```yaml
    type: custom:button-card
    grid_options:
      rows: 2
      columns: 6
    section_mode: true
    update_timer: 500ms
    variables:
      throttle: |
        [[[ return window.uixCoordinator?.hassThrottleEnable; ]]]
      throttleMs: |
        [[[ return window.uixCoordinator?.hassThrottleMs; ]]]
    styles:
      card:
        - "--throttle-icon-color": >
            [[[ return variables.throttle ? "var(--state-active-color)" :
            "var(--state-inactive-color)"; ]]]
        - "--ha-ripple-hover-color": >
            [[[ return variables.throttle ? "var(--state-active-color)" :
            "var(--state-inactive-color)"; ]]]
    name: >
      [[[ return variables.throttle ? `Throttle ON (${variables.throttleMs}ms)` :
      "Throttle OFF"; ]]]
    icon: >
      [[[ return variables.throttle ? "mdi:speedometer" : "mdi:speedometer-slow";
      ]]]
    color: var(--throttle-icon-color)
    tap_action:
      action: javascript
      javascript: >
        [[[ variables.throttle ? window.uixCoordinator?.setThrottleOverride({
        enable: false }) : window.uixCoordinator?.setThrottleOverride({ enable: true
        }); ]]]
    ```

## Référence de configuration

| Réglage | Valeur par défaut | Description |
|---|---|---|
| Throttle entity state updates | Désactivé | Active ou désactive globalement la limitation des mises à jour d'état. |
| Throttle interval | 200 ms | Délai minimal entre les rendus déclenchés par un changement d'état. Plage : 50 à 10 000 ms. |

## Quand utiliser la limitation

Elle est utile lorsque :

- des **entités qui changent rapidement** (RSSI Bluetooth, capteurs d'énergie ou météo, par exemple) rendent le tableau de bord lent ou provoquent des scintillements ;
- des **appareils lents ou peu puissants** (tablettes murales ou anciens navigateurs) ont du mal à suivre les rendus fréquents ;
- des **tableaux de bord complexes**, comportant de nombreuses cartes ou beaucoup de styles UIX, ralentissent lors de fortes rafales de mises à jour.

!!! warning
    Un intervalle trop élevé peut rendre le tableau de bord moins réactif. Une valeur de 200 à 500 ms constitue un bon point de départ. Le mécanisme de rattrapage garantit l'affichage du dernier état, même si les mises à jour intermédiaires sont ignorées. Si vous basculez rapidement un capteur, la seconde mise à jour apparaîtra au plus tard à la fin de l'intervalle, plus 50 ms.
