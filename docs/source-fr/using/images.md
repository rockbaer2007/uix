---
description: Découvrez comment styliser les images d'entités.
---
# Styliser les images d'entités

UIX peut remplacer l'image d'entité affichée en arrière-plan par les éléments suivants :

- `ha-entity-marker` (marqueurs d'entités de la carte)
- `ha-tile-icon` (icônes de carte Tuile)
- `state-badge` (badges d'état)
- `ha-user-badge` (badges utilisateur)
- `ha-person-badge` (badges de personne)
- `hui-entity-badge` (badge d'entité)

Vous pouvez appliquer le style par une [surcharge propre à une entité](#specifying-for-an-entity-override) ou par une [surcharge générique](#specifying-generic-override).

!!! note
    Pour styliser l'image d'un badge d'entité (`hui-entity-badge`), activez **Afficher l'image de l'entité** (`show_entity_picture: true` en YAML).

## Définir une surcharge pour une entité { #specifying-for-an-entity-override }

Définissez une variable CSS sous la forme `--uix-image-for-<entity_id>`, en remplaçant chaque `.` de l'identifiant d'entité par `_`. Lorsqu'un élément est affiché pour l'entité correspondante, l'image de fond est remplacée par l'URL fournie.

Les modèles sont pris en charge.

```yaml
type: tile
entity: person.jim
uix:
  style: |
    :host {
      --uix-image-for-person_jim: /local/photos/jim.jpg;
    }
```

!!! tip
    - La variable peut être définie à n'importe quel niveau parent du DOM. UIX la détecte sur l'élément à partir des styles calculés. Si elle n'est pas définie ou que l'entité ne correspond pas, l'image d'origine reste inchangée.
    - Pour appliquer une surcharge à toute l'interface Home Assistant, ajoutez `--uix-image-for-<entity_id>` aux variables de thème `uix-root(-yaml)`, `uix-config(-yaml)` et `uix-more-info(-yaml)`.

## Définir une surcharge générique { #specifying-generic-override }

Définissez la variable CSS générique `--uix-image` dans le contexte de l'image à remplacer, par exemple sur un élément contenant `ha-entity-marker` (par exemple une carte), `ha-tile-icon` (par exemple une carte Tuile) ou `state-badge` (par exemple une ligne Entités).

Lorsqu'un élément pris en charge est affiché dans ce contexte, l'image de fond est remplacée par l'URL fournie, quelle que soit l'entité.

Les modèles sont pris en charge.

!!! tip
    Si `--uix-image` et `--uix-image-for-<entity_id>` sont tous deux définis, `--uix-image` est prioritaire.
