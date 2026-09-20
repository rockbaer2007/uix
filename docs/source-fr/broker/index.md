---
title: UIX Broker
description: Créez des interactions déclaratives d'événements Frontend pour Home Assistant avec UIX Broker.
---
# UIX Broker

UIX Broker transforme les événements du navigateur, les raccourcis clavier et les événements du bus Home Assistant en interactions déclaratives. Une interaction sélectionne un élément, vérifie des règles optionnelles, puis exécute les directives dans l'ordre configuré.

```text
Realm → Listen → Interaction anchor → Rules (Optional anchors) → Directives (Optional anchors)
```

Utilisez UIX Broker lorsqu'un comportement d'interface peut être configuré au lieu d'être écrit sous forme de carte personnalisée, script ou patch. UIX Broker peut réagir à un clic, modifier un événement avant de le redistribuer, placer le focus sur un élément, mettre à jour une propriété, appeler une méthode sûre, ajouter un bouton interactif et exécuter des actions JavaScript avec les variables de l'interaction.

```yaml
uix_broker:
  - realm: browser
    listen: click
    anchor: target
    rules:
      - ".action-button"
    directives:
      - type: block
      - type: event
        name: another-action
        data:
          source: action-button
```

## Guides UIX Broker

- [Broker](./broker.md) — structure des interactions, sources de configuration, cycle de vie et débogage.
- [Contextes](./realms.md) — événements du navigateur, raccourcis clavier et bus d'événements Home Assistant.
- [Ancres d'interaction](./interaction-anchors.md) — sélection avec le chemin d'événement composé et `select_tree`.
- [Règles](./rules.md) — correspondance d'élément hôte, de données capturées et d'identité de navigateur.
- [Directives](./directives.md) — `block`, `property`, `event`, `call`, `button` et actions Home Assistant.
- [Exemples](./examples.md) — exemples. Consultez aussi [UIX Guides](https://uix-guides.lf.technology), où d'autres exemples détaillés peuvent être publiés.

!!! note
    Pour la correspondance d'identité de navigateur, [Browser Mod](https://github.com/thomasloven/hass-browser_mod) est requis.

## Fonctions prévues

UIX Broker est en cours de développement actif. Les fonctions et exemples actuels proviennent d'idées partagées par la communauté. Si vous avez une idée pour étendre UIX Broker, ouvrez une [discussion GitHub](https://github.com/Lint-Free-Technology/uix/discussions). Les fonctions qui reçoivent 10 votes positifs peuvent devenir une demande de fonctionnalité dans le suivi GitHub de UIX.

Les fonctions UIX Broker prévues comprennent :

- **Règle JavaScript** : exécute du JavaScript avec l'état actuel de l'interaction fourni comme variables. Elle renvoie un objet `{result: <truthy>, [optional] namedObject: <object data>}` ; l'objet optionnel `namedObject` devient alors disponible pour les règles et directives suivantes.
- **Directive d'action JavaScript étendue** : prend en charge un retour de l'action JavaScript actuelle. Le format est `{continue: <truthy>, [optional] namedObject: <object data>}`. Si `continue` est faux, les directives suivantes ne sont pas exécutées. L'objet optionnel devient disponible pour le reste des directives.
- **Règle de modèle Jinja2** : rend un modèle Jinja2 ponctuel qui renvoie un résultat véridique et peut aussi renvoyer des données objet.
