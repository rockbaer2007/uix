---
title: Désactiver la variable de modèle hash et ses mises à jour
description: Découvrez comment désactiver les mises à jour de la variable hash et quand utiliser cette option de performance.
---
# Désactiver la variable de modèle hash et ses mises à jour

Par défaut, UIX expose `hash` comme variable de modèle et met à jour les modèles lorsque le fragment d'URL (`#...`) change. Cette option désactive ce comportement pour éviter les nouvelles liaisons et mises à jour Forge déclenchées par le hash.

## Réglage depuis l'interface de l'intégration

Cette option est **désactivée par défaut**. Pour l'activer :

1. Dans Home Assistant, ouvrez **Paramètres → Appareils et services → UI eXtension → Configurer**.
2. Sélectionnez **Réglages de performance**.
3. Activez **Désactiver la variable de modèle hash et ses mises à jour**.
4. Enregistrez.

Le réglage prend effet immédiatement dans toutes les sessions de navigateur connectées ; aucun rechargement n'est nécessaire.

## Comportement lorsqu'elle est activée

Lorsque cette option est activée :

- La variable de modèle `hash` n'est **pas disponible**.
- Les changements d'URL limités au hash ne déclenchent **pas** de nouvelle liaison des modèles UIX.
- Les changements d'URL limités au hash ne déclenchent **pas** de mise à jour UIX Forge.

!!! warning
    Tout modèle qui référence `hash` sans valeur par défaut génère une erreur lorsque cette option est activée, car la variable n'est plus disponible. Si vous souhaitez pouvoir activer ou désactiver l'option, définissez une valeur par défaut, par exemple `{{ hash | default("") }}`, ou utilisez une variable locale dans les modèles plus complexes : `{% set hashWithDefault = hash | default("") %}`.
