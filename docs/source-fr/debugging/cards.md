# Débogage des cartes

La navigation dans le DOM peut être difficile à comprendre au début, mais elle devient vite plus claire avec la pratique.

Utilisez l'inspecteur d'éléments de votre navigateur pour voir les étapes suivies par UIX.

- Ouvrez l'inspecteur et trouvez l'élément de base, par exemple `#shadow-root`, une carte dans `<hui-card>` ou un `<ha-card>` dans une carte personnalisée. Consultez [Concepts - application](../concepts/application.md). Il contient un élément `<uix-node>`, même sans style défini.
- Vérifiez que l'élément `<uix-node>` est sélectionné.
- Ouvrez la console du navigateur. Dans Chrome, `Esc` ouvre simultanément la console et l'inspecteur.
- Saisissez `$0.uix_input`, puis validez. Cette propriété contient le style reçu par cette étape de la chaîne. Une chaîne indique la fin de la chaîne ; un objet permet de continuer.
- Saisissez `$0.uix_children`, puis validez. Cet ensemble contient les `<uix-node>` de l'étape suivante. Cliquez sur « uix » dans la valeur pour sélectionner l'élément correspondant et poursuivre l'inspection.
- Utilisez aussi `$0.uix_parent` pour trouver le parent d'un `<uix-node>` dans la chaîne.

Pour obtenir davantage d'informations, ajoutez ceci à la configuration de la carte concernée :

```yaml
uix:
  debug: true
```

## Définir le débogage avec les variables de thème

Comme `uix:` → `debug: true` sur une carte, le débogage peut aussi être activé par une variable de thème. C'est parfois la seule façon de déboguer un type ou une classe lors du style d'un panneau qui n'est pas un tableau de bord Lovelace.

Vous pouvez activer le débogage de deux façons :

1. Définissez `uix-<type>-debug: true` dans le YAML du thème, sans les deux tirets initiaux, pour déboguer tous les éléments de type `<type>`. En CSS, la variable est `--uix-<type>-debug`.
2. Définissez `uix-<type>-<class>-debug: true` pour les éléments de type `<type>` ayant la classe `<class>`. En CSS, utilisez `--uix-<type>-<class>-debug`. Cela inclut les classes créées par UIX et celles définies dans la configuration d'une carte ou d'un élément.

Exemple :

```yaml
my-awesome-theme:
  uix-theme: my-awesome-theme

  uix-card-debug: true # Débogue tous les éléments UIX de type `card`
```

```yaml
my-awesome-theme:
  uix-theme: my-awesome-theme

  uix-card-type-energy-sankey-debug: true # Débogue les cartes avec la classe UIX 'type-energy-sankey'
  uix-badge-my-class-debug: true # Débogue les badges ayant my-class dans la configuration UIX
```

!!! warning "Définissez les variables dans le thème Home Assistant"
    Les variables de débogage sont définies dans le thème Home Assistant actif, global ou appliqué localement à une vue ou une carte. Elles ne sont **pas** lues depuis le thème `uix-theme`. Si les deux thèmes sont différents, définissez les variables de débogage dans le thème Home Assistant actif.

    ```yaml
    theme-mods:
      ... UIX theme variables, styles, macros go here ...
    
    my-awesome-theme:
      uix-card-type-energy-sankey-debug: true # Debug card which has uix class 'type-energy-sankey'
      uix-badge-my-class-debug: true # Debug badges which have my-class set by uix config

      uix-theme: theme-mods
    ```
