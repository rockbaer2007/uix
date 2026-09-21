---
title: Désactiver la correction du style des icônes
description: Découvrez comment désactiver la correction du style des icônes et quand utiliser cette option de performance.
---
# Désactiver la correction du style des icônes

Par défaut, UIX corrige les éléments d'icône standard de Home Assistant (`ha-icon`, `ha-state-icon`, `ha-svg-icon`) pour permettre les styles et surcharges personnalisés, comme `--uix-icon`, `--uix-icon-color`, `--uix-icon-dim` ou `--uix-icon-for-<entity_id>`. UIX propose une option pour désactiver cette correction afin d'améliorer le rendu et de réduire la charge processeur des appareils peu puissants.

## Réglage depuis l'interface de l'intégration

Cette option est **désactivée par défaut**. Pour l'activer :

1. Dans Home Assistant, ouvrez **Paramètres → Appareils et services → UI eXtension → Configurer**.
2. Sélectionnez **Réglages de performance**.
3. Activez **Désactiver la correction du style des icônes**.
4. Enregistrez.

Le réglage prend effet immédiatement dans toutes les sessions de navigateur connectées ; aucun rechargement n'est nécessaire.

## Comportement lorsqu'elle est activée

Lorsque cette option est activée :

- Les éléments d'icône standard Home Assistant ne sont **pas corrigés** ni surveillés pour les propriétés d'icône personnalisées.
- Les surcharges existantes d'icône ou de couleur via des variables CSS, par exemple `--uix-icon`, `--uix-icon-color`, `--uix-icon-dim` ou `--uix-icon-for-*`, ne s'appliquent **pas** aux éléments d'icône standard.
