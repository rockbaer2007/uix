---
title: Clearing cache
---
# Vider le cache de l'interface Home Assistant

Si vous devez vider le cache de l'application Frontend Home Assistant, utilisé en plus du cache du navigateur, utilisez une action personnalisée pour vider ce cache et recharger le navigateur. C'est particulièrement utile sur les appareils où l'option est cachée dans un menu de débogage. L'action efface aussi davantage que le cache Frontend, par exemple localStorage et des données telles que l'identifiant Browser Mod.

Consultez l'action UIX [`clear_cache`](../extras/uix-actions.md#clear-cache-clearing-home-assistant-frontend-cache).
