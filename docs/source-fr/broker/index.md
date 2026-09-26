---
title: UIX Broker
description: Créez des interactions déclaratives d'événements frontend pour Home Assistant avec UIX Broker.
---
# UIX Broker

UIX Broker transforme les événements du navigateur, les raccourcis clavier et les événements du bus Home Assistant en interactions déclaratives. Une interaction sélectionne un élément du navigateur, vérifie les règles facultatives, puis exécute les directives dans leur ordre configuré. La directive `block` fait exception : elle bloque synchroniquement l'événement déclencheur avant l'exécution des autres directives.

```text
Realm → Listen → Interaction anchor → Rules (Optional anchors) → Directives (Optional anchors)
```

Utilisez UIX Broker lorsqu'un comportement d'interface peut être configuré plutôt que codé dans une carte, un script ou un correctif personnalisé. UIX Broker peut réagir à un clic, modifier un événement avant de le redistribuer, placer le focus sur un élément, mettre à jour une propriété d'objet et appeler une méthode d'élément sécurisée. Il peut aussi ajouter des boutons, des badges, du texte, des icônes de tuile, des info-bulles et des demandes de déverrouillage ; associer ou exécuter des actions Home Assistant ; afficher des modèles ; évaluer du JavaScript ; et insérer des pauses entre les opérations.

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
- [Règles](./rules.md) — correspondance de l'élément hôte, des données capturées, de l'identité du navigateur, de l'utilisateur, du statut d'administrateur, du fragment d'URL, des paramètres de recherche et du panneau.
- [Directives](./directives.md) — `block`, `property`, `event`, `call`, `button`, `badge`, `text-content`, `tile-icon`, `tooltip`, `lock`, `action-handler`, `action`, `template`, `javascript` et `wait`.
- [Exemples](./examples.md) — exemples. Voir également [UIX Guides](https://uix-guides.lf.technology), où des exemples plus détaillés peuvent être publiés.

!!! note
    Pour la correspondance de l'identité du navigateur, [Browser Mod](https://github.com/thomasloven/hass-browser_mod) est requis.
