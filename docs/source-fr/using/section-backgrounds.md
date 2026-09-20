---
description: Découvrez comment styliser la couleur et l'opacité de l'arrière-plan des sections.
---
# Styliser les arrière-plans de section

L'arrière-plan d'une section est un élément frère de la section. Il ne peut donc pas être ciblé directement par UI eXtension depuis la section. Deux possibilités existent : une méthode simple avec des variables UIX dédiées à la couleur et à l'opacité, ou un style direct de l'arrière-plan avec UIX.

## Option 1 : utiliser les variables CSS UIX

UIX peut styliser la couleur et l'opacité de l'arrière-plan d'une section avec les variables CSS `--uix-section-background-color` et `--uix-section-background-opacity`, appliquées à la section ou à son parent. Comme pour les autres styles CSS UIX, les modèles sont pris en charge.

!!! info
    L'arrière-plan `hui-section-background` est un élément frère de la section. Définir `--ha-section-background-color` dans le style UIX de la section ne s'applique donc pas. À chaque mise à jour, UIX applique directement `--section-background-color` et `--section-background-opacity` à `hui-section-background`.

Pour appliquer l'arrière-plan, la configuration de la section doit l'activer. La forme courte minimale prise en charge par Home Assistant est `background: true` ; elle ajoute `hui-section-background` avec les valeurs par défaut de couleur et d'opacité.

!!! example
    ```yaml
    type: grid
    cards: []
    background: true
    uix:
      style: |
        :host {
            --uix-section-background-color: {{ 'red' if is_state('input_boolean.test_boolean', 'on') else 'green' }};
            --uix-section-background-opacity: {{ states('input_number.increment_test') | float / 100 }};
        }
    ```

## Option 2 : ajouter le style UIX à la configuration d'arrière-plan de la section

Vous pouvez ajouter des options de style UIX à la configuration de l'arrière-plan. Elles cibleront l'élément d'arrière-plan. Toute la configuration de la section est disponible dans les modèles.

Pour la couleur et l'opacité, il est préférable de définir `--ha-section-background-color` et `--ha-section-background-opacity`. Vous pouvez aussi styliser directement l'arrière-plan et son opacité, mais vous devrez utiliser `!important`.

!!! example
    ```yaml
    type: grid
    cards: []
    background:
      uix:
        style: |
          :host {
            --ha-section-background-color: yellow;
            border: 2px solid red;
            box-shadow: 10px 5px 5px red;
          }
    ```
