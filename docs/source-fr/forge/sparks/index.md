---
title: Sparks
description: Comportements autonomes qui enrichissent un élément créé avec UIX Forge.
---
# Sparks

Les sparks sont des comportements facultatifs que vous ajoutez à un élément créé avec [UIX Forge](../index.md), dans la liste `forge.sparks`. Chaque spark possède une clé `type` et ses propres options de configuration.

Sparks disponibles :

- :speech_balloon: [Info-bulle](./tooltip.md) — ajoute une info-bulle stylisée à un élément de l'élément créé.
- :material-button-cursor: [Bouton](button.md) — insère un élément interactif `ha-button` avant ou après un élément cible.
- :label: [Attribut](attribute.md) — ajoute, remplace ou supprime un attribut d'un élément cible.
- :zap: [Événement](event.md) — reçoit les événements DOM des actions `fire-dom-event` et expose leurs données sous forme de variables de modèle.
- :star: [Icône de tuile](tile-icon.md) — insère un élément `ha-tile-icon` avant ou après un élément cible.
- :shield: [Badge d'état](state-badge.md) — insère un élément `state-badge` avant ou après un élément cible.
- :material-grid: [Grille](grid.md) — applique une disposition CSS Grid à un conteneur.
- :mag: [Recherche](search.md) — recherche des éléments par sélecteur CSS et filtre textuel facultatif, puis modifie leurs classes, attributs ou contenu textuel.
- :material-map: [Carte](map.md) — conserve le niveau de zoom et le centre d'une carte lors des mises à jour d'état de Home Assistant.
- :material-lock: [Verrou](lock.md) — affiche une icône de verrouillage et bloque les interactions jusqu'à la saisie d'un code PIN, d'une phrase secrète ou à une confirmation.
- :material-information-outline: [Plus d'informations](more-info.md) — intègre les informations détaillées Home Assistant d'une entité.
- :material-star-four-points-outline: [Icône superposée](overlay-icon.md) — superpose une `ha-icon` ou une `ha-state-icon` à un élément.
- :material-image-outline: [Arrière-plan](background.md) — ajoute derrière un élément un arrière-plan coloré, une image, une vidéo ou un flux de caméra.
- :material-palette: [Thème](theme.md) — applique un thème frontend à l'élément ou à l'un de ses descendants.
