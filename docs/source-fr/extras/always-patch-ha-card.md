---
title: Toujours corriger ha-card (expérimental)
description: Découvrez comment activer en permanence la correction de ha-card avec ce réglage expérimental.
---
# Toujours corriger ha-card

Par défaut, UIX ne corrige pas `ha-card` s'il ne trouve pas de configuration de carte dans le premier élément Frontend ou personnalisé de son arbre DOM parent. Cette option expérimentale permet de toujours corriger l'élément ha-card afin que la variable de thème `uix-card(-yaml)` puisse s'appliquer. Un `ha-card` sans configuration peut être utilisé dans des panneaux de configuration ou personnalisés.

Lorsqu'un `ha-card` est corrigé sans configuration, la classe `type-generic-card` lui est ajoutée.

## Réglage depuis l'interface de l'intégration

Cette option est **désactivée par défaut**. Pour l'activer :

1. Dans Home Assistant, ouvrez **Paramètres → Appareils et services → UI eXtension → Configurer**.
2. Sélectionnez **Réglages expérimentaux** dans le menu.
3. Activez **Toujours corriger ha-card**.
4. Enregistrez.

Le réglage est disponible immédiatement dans toutes les sessions de navigateur connectées. Un rechargement de la page peut être nécessaire pour qu'il prenne effet.

## Comportement lorsqu'elle est activée

Lorsque cette option est activée :

- `ha-card` est toujours corrigé, même sans configuration de carte disponible.
- Dans ce cas, la classe `type-generic-card` est ajoutée à `ha-card`.
