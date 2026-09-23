---
title: UIX Broker
description: Créez des interactions déclaratives d'événements frontend pour Home Assistant avec UIX Broker.
---
# UIX Broker

UIX Broker transforme les événements du navigateur, les raccourcis clavier et les événements de bus d'événements Home Assistant en interactions déclaratives. Une interaction sélectionne un élément du navigateur, vérifie les règles facultatives, puis exécute les directives dans leur ordre configuré.

```text
Realm → Listen → Interaction anchor → Rules (Optional anchors) → Directives (Optional anchors)
```

Utilisez UIX Broker lorsqu'un comportement d'interface peut être configuré plutôt que écrit sous forme de carte, de script ou de correctif personnalisé. UIX Broker peut réagir à un clic, personnaliser un événement avant de le redistribuer, concentrer un élément, mettre à jour une propriété d'objet, appeler une méthode d'élément sécurisée, ajouter un bouton interactif et exécuter des actions JavaScript avec des variables d'interaction disponibles.

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

## Guides du courtier UIX

- [Broker](./broker.md) — structure d'interaction, sources de configuration, cycle de vie et débogage.
- [Domaines](./realms.md) — événements de navigateur, raccourcis clavier et événements de bus d'événements Home Assistant.
- [Ancres d'interaction](./interaction-anchors.md) — sélection du chemin d'événement composé et de l'élément `select_tree`.
- [Règles](./rules.md) — correspondance de l'élément hôte, des données capturées et de l'identité du navigateur.
- [Directives](./directives.md) — `block`, `property`, `event`, `call`, `button` et actions Home Assistant.
- [Exemples](./examples.md) — exemples. Voir également [UIX Guides](https://uix-guides.lf.technology), où des exemples plus détaillés peuvent être publiés.

!!! note
    Pour la correspondance de l'identité du navigateur, [Browser Mod](https://github.com/thomasloven/hass-browser_mod) est requis.

## Fonctionnalités futures

UIX Broker est en développement actif. Jusqu'à présent, toutes les fonctionnalités et exemples proviennent d'idées d'utilisateurs partagées sur le forum de la communauté. Si vous avez une idée sur la façon dont UIX Broker peut être étendu, veuillez démarrer une [discussion GitHub](https://github.com/Lint-Free-Technology/uix/discussions). Les fonctionnalités qui obtiennent 10 votes positifs peuvent être déplacées vers une demande de fonctionnalité dans le suivi des problèmes UIX GitHub.

Les futures fonctionnalités prévues d'UIX Broker incluent :

- **Règle JavaScript** : exécute JavaScript avec l'état d'interaction actuel fourni sous forme de variables. Renvoie un objet avec `{result: <truthy>, [optional] namedObject: <object data>}`, avec le `namedObject` facultatif alors disponible pour d'autres règles et toutes les directives.
- **Directive d'action JavaScript étendue** : prend en charge le retour de la directive d'action JavaScript actuelle. Format de retour : `{continue: <truthy>, [optional] namedObject: <object data>}`. Si `continue` est faux, aucune autre directive n'est exécutée. Le `namedObject` optionnel est alors disponible pour le reste des opérations de directive.
- **Règle du modèle Jinja2** : restitue un modèle Jinja2 unique qui renvoie un résultat véridique et peut éventuellement renvoyer des données d'objet.
