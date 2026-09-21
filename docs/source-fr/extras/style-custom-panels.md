---
title: Styliser les panneaux personnalisés chargés dans une iframe (expérimental)
description: Découvrez comment activer le style des panneaux personnalisés chargés dans une iframe.
---
# Styliser les panneaux personnalisés chargés dans une iframe

Par défaut, UIX ne stylise pas les panneaux personnalisés chargés dans une iframe. Utilisez ce réglage expérimental pour activer leur style. Consultez [Styliser les panneaux personnalisés dans une iframe](../using/custom-panels.md) pour plus d'informations et d'exemples.

## Réglage depuis l'interface de l'intégration

Cette option est **désactivée par défaut**. Pour l'activer :

1. Dans Home Assistant, ouvrez **Paramètres → Appareils et services → UI eXtension → Configurer**.
2. Sélectionnez **Réglages expérimentaux**.
3. Activez **Styliser les panneaux personnalisés chargés dans une iframe**.
4. Enregistrez.

Le réglage est disponible immédiatement dans toutes les sessions de navigateur connectées. Un rechargement peut être nécessaire pour l'appliquer au panneau personnalisé actuellement affiché.

## Comportement lorsqu'elle est activée

Lorsque cette option est activée :

- les panneaux personnalisés sont stylisés par une correction dans `ha-panel-custom`. Elle crée un fichier Frontend `customPanelJS` corrigé pour l'iframe, qui exécute le `customPanelJS` standard de Home Assistant puis un module JavaScript UIX condensé.
- le style UIX s'applique à l'élément principal du panneau personnalisé.
- si UIX détecte qu'aucun thème n'est appliqué, UIX Styling utilise le thème Frontend Home Assistant actuellement chargé. Certains panneaux, comme HACS, appliquent déjà le thème : le style UIX l'hérite alors.
