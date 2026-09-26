---
description: Apprenez à styliser les panneaux d'applications et d'ingress de Home Assistant.
---
# Styliser les panneaux d'applications et d'ingress

Home Assistant affiche la page d'une application ou d'un ingress dans un élément `<ha-panel-app>`. Utilisez la clé de thème `uix-app` ou `uix-app-yaml` pour styliser la racine shadow DOM ouverte de ce panneau.

Il ne s'agit pas de la liste des applications installées à `/config/apps/installed`, qui appartient au panneau de configuration et utilise `uix-config`.

## Exemple

Cet exemple stylise l'en-tête du panneau Home Assistant et place une superposition CRT non interactive au-dessus de l'iframe d'ingress. Il applique également des styles au document Zigbee2MQTT chargé dans cette iframe.

Avant d'utiliser le bloc `uix-zigbee2mqtt`, activez l'option expérimentale [Style des panneaux dans les iFrames](https://uix.lf.technology/extras/style-frame-panels/).

```yaml
Mon thème :
  uix-theme: Mon thème

  uix-app: |
    :host {
      position: relative;
    }

    .header {
      background: #041b0b !important;
      color: #7cff88 !important;
    }

    :host::after {
      content: "";
      position: absolute;
      inset: 0;
      z-index: 1;
      pointer-events: none;
      background: repeating-linear-gradient(
        to bottom,
        rgb(124 255 136 / 8%) 0,
        rgb(124 255 136 / 8%) 1px,
        transparent 1px,
        transparent 3px
      );
    }

  # Ce bloc cible l'iframe Zigbee2MQTT, pas ha-panel-app.
  uix-zigbee2mqtt: |
    :root {
      --color-base-100: #041b0b;
      --color-base-content: #7cff88;
      --bg-color: #041b0b;
    }
```

## Portée

!!! info
    Le style du contenu des iFrames d'applications de même origine est disponible depuis la version 3.4.0-beta.1.

`uix-app` continue de styliser l'habillage du panneau Home Assistant et peut superposer son iframe. UIX installe également son moteur interne dans les iFrames d'applications de même origine. Le style du contenu de l'iframe nécessite l'option expérimentale [Style des panneaux dans les iFrames](https://uix.lf.technology/extras/style-frame-panels/) ; le style du panneau hôte ne la nécessite pas.

Pour le contenu de l'iframe, utilisez `uix-<app-slug>` (ou sa forme `-yaml`). UIX vérifie d'abord le slug complet de l'application Home Assistant, puis un slug indépendant du dépôt après suppression de `core_`, `local_` ou d'un hash de dépôt de huit caractères. Par exemple, `uix-a0d7b954_nodered` est prioritaire sur `uix-nodered`.

!!! tip
    Pour trouver le slug d'application à utiliser avec `uix-<app-slug>`, consultez les informations UIX dans la console développeur du navigateur. Elles sont produites par `uixFrame.js` et ressemblent à ceci :

    <span style="background:#CE3226;color:white;padding:2px 5px;font-weight:bold;border-radius:5px;">💡 UIX 8.4.0 EST INSTALLÉ 💡 pour 45df7312_zigbee2mqtt</span>

    La dernière partie de ce message est le slug de l'application, par exemple `45df7312_zigbee2mqtt`.

Le moteur de rendu des iFrames est une API interne partagée avec les panneaux personnalisés dans des iFrames. Les notions visibles par l'utilisateur restent distinctes : `uix-app` désigne toujours le conteneur `<ha-panel-app>`, tandis que `uix-panel-custom` désigne toujours le conteneur `<ha-panel-custom>`.

## Style des cartes

Certains panneaux d'applications, comme KNX Frontend, utilisent `ha-card` pour afficher leurs informations. Pour que le thème `uix-card(-yaml)` s'y applique, activez l'option expérimentale [Toujours corriger ha-card](https://uix.lf.technology/extras/always-patch-ha-card/).
