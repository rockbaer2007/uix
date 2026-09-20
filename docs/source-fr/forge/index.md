---
title: UIX Forge
description: Découvrez UIX Forge, un élément personnalisé puissant qui combine modèles, sparks et UIX Styling.
---
UIX Forge permet de créer des éléments Home Assistant avec des modèles pour toute leur configuration, ainsi que des extensions avancées grâce aux [Sparks UIX Forge](./sparks/).

Les éléments Home Assistant pris en charge sont les cartes, badges, lignes, sections et éléments image. Les moules intercontextes permettent d'intégrer un type d'élément dans un autre contexte parent, par exemple une carte utilisée comme ligne dans une carte Entités ; consultez les [moules intercontextes](./forge.md#cross-context-molds).

Consultez [Forge](./forge.md) pour la référence complète de la configuration.

## Fonderies

Une **fonderie** est un modèle UIX Forge enregistré sur le serveur. Elle permet de définir une seule fois des configurations `forge`, `element` et `uix` réutilisables sur plusieurs cartes. Référencez une fonderie avec la clé `foundry:` et ne remplacez localement que ce qui est nécessaire.

Consultez [Fonderies](./foundries.md) pour un guide complet sur la fusion, les fonderies imbriquées et leur gestion dans les options de l'intégration.

## Sparks

Les Sparks sont des comportements optionnels ajoutés à la liste `forge.sparks`. Chaque Spark possède une clé `type` et ses propres options.

Sparks disponibles :

- :speech_balloon: [Infobulle](./sparks/tooltip.md) — attache une infobulle stylisée à tout élément créé.
- :material-button-cursor: [Bouton](./sparks/button.md) — attache un bouton stylisé (`ha-button`) avec actions avant ou après un élément.
- :label: [Attribut](./sparks/attribute.md) — ajoute, remplace ou supprime un attribut d'un élément.
- :zap: [Événement](./sparks/event.md) — reçoit des événements DOM d'actions `fire-dom-event` et expose leurs données comme variables de modèle.
- :star: [Icône Tile](./sparks/tile-icon.md) — insère un élément `ha-tile-icon` avant ou après un élément.
- :shield: [Badge d'état](./sparks/state-badge.md) — insère un élément `state-badge` avant ou après un élément.
- :material-grid: [Grille](./sparks/grid.md) — applique une mise en page **CSS Grid** à un conteneur.
- :mag: [Recherche](./sparks/search.md) — recherche des éléments avec un sélecteur CSS et applique des modifications aux éléments trouvés.
- :material-map: [Carte](./sparks/map.md) — conserve le niveau de zoom et le centre d'une carte lors des mises à jour Home Assistant.
- :material-lock: [Verrou](./sparks/lock.md) — superpose un verrou et bloque l'interaction jusqu'à la saisie d'un PIN, d'une phrase secrète ou d'une confirmation.
- :material-star-four-points-outline: [Icône superposée](./sparks/overlay-icon.md) — superpose une `ha-icon` ou `ha-state-icon` à un élément.
- :material-image-outline: [Arrière-plan](./sparks/background.md) — ajoute un calque d'arrière-plan, couleur, image, vidéo ou caméra, derrière un élément.
- :material-palette: [Thème](./sparks/theme.md) — applique un thème Frontend à l'élément créé ou à l'un de ses descendants.
