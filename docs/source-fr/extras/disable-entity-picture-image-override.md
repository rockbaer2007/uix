---
title: Désactiver les surcharges d'image des entités
description: Découvrez comment désactiver les surcharges d'image des entités et quand utiliser cette option de performance.
---
# Désactiver les surcharges d'image des entités

Par défaut, UIX corrige les badges et marqueurs standard de Home Assistant (`ha-entity-marker`, `ha-tile-icon`, `state-badge`, `ha-user-badge`, `ha-person-badge`) afin de permettre les surcharges d'image personnalisées comme `--uix-image` ou `--uix-image-for-<entity_id>`. Cette option désactive cette correction pour améliorer le rendu sur les appareils peu puissants.

## Réglage depuis l'interface de l'intégration

Cette option est **désactivée par défaut**. Pour l'activer :

1. Dans Home Assistant, ouvrez **Paramètres → Appareils et services → UI eXtension → Configurer**.
2. Sélectionnez **Réglages de performance**.
3. Activez **Désactiver les surcharges d'image des entités**.
4. Enregistrez.

Le réglage prend effet immédiatement dans toutes les sessions de navigateur connectées ; aucun rechargement n'est nécessaire.

## Comportement lorsqu'elle est activée

Lorsque cette option est activée :

- Les badges et marqueurs standard ne sont **pas corrigés** ni surveillés pour les propriétés d'image personnalisées.
- Les surcharges d'image existantes via des variables CSS, par exemple `--uix-image` ou `--uix-image-for-*`, ne s'appliquent **pas** aux badges ou marqueurs.
