---
description: Découvrez d'autres techniques de style, dont l'utilisation de mod-card pour les cartes personnalisées difficiles à traiter.
---
# Autres styles

Les cartes qui ne possèdent pas de `<ha-element>` peuvent tout de même être stylisées avec la carte `custom:mod-card` fournie. Cette solution n'est nécessaire que dans de **très rares** cas et peut créer davantage de problèmes qu'elle n'en résout. Votre carte contient probablement une autre carte : appliquez alors le style à **celle-ci**.

??? warning "Utilisez custom:mod-card avec prudence"
    ```yaml
    type: custom:mod-card
    card:
      type: custom:beloved-custom-card
      ...
    uix:
      style: |
        ha-card {
          ...
        }
    ```
    La mod-card crée un élément `<ha-card>` et y place votre carte. Cette carte sera transparente, sans bordure ni arrière-plan.
