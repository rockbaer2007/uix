---
description: Réponses aux questions fréquentes sur UI eXtension.
hide:
  - toc
  - navigation
---
# FAQ

## Comment migrer au mieux depuis Card-mod ?

- Désinstallez Card-mod.
- Si vous utilisez `extra_module_url` pour charger la ressource Card-mod, supprimez cette entrée et redémarrez Home Assistant.
- Suivez ensuite le [guide de démarrage rapide de UI eXtension](./quick-start.md).

!!! tip "Ajouter UI eXtension comme service"
    UI eXtension est une intégration. Après l'avoir installée avec HACS, vous devez l'ajouter comme service. Ne sautez pas l'étape **Ajouter le service UI eXtension** ; elle peut facilement passer inaperçue si vous n'avez pas l'habitude d'ajouter des intégrations comme services.

??? warning "UI Lovelace Minimalist peut charger la ressource Card-mod"
    Si UI Lovelace Minimalist est installé, il peut charger la ressource Card-mod. En raison de l'ordre de chargement des intégrations, UI eXtension ne peut pas détecter ce conflit. Pour continuer à utiliser UI Lovelace Minimalist avec UI eXtension, désactivez l'option `Include custom card resources it's depending on` dans sa configuration, puis chargez les cartes personnalisées nécessaires avec HACS ou manuellement.

    ![UI Minimalist custom card option](../assets/page-assets/faq/ui-minimalist-option.png)

## UI eXtension peut-il remplacer Card-mod directement ?

Oui. UI eXtension remplace directement Card-mod jusqu'à la version 4.2.1 et prend en charge les configurations de cartes et de thèmes Card-mod. Il est conseillé d'adopter `uix:` dans les cartes et `uix-<thing>(-yaml)` dans les thèmes, mais ce changement n'est pas obligatoire.

## UI eXtension est-il simplement Card-mod avec une documentation différente ?

Non. Le code de UI eXtension a évolué pour faire de UIX le domaine et la clé de configuration principaux. Les clés Card-mod restent prises en charge, mais `uix:` est prioritaire.

??? info "Différences entre UI eXtension et Card-mod"
    - La clé de configuration des cartes est `uix:`.
    - La clé des thèmes est `uix-theme:`.
    - Les clés de thème `thing` sont `uix-<thing>(-yaml):`.
    - Le nœud HTML de UI eXtension est `<uix-node>` ; ses propriétés se rapportent toutes à `uix`.
    - Vous pouvez utiliser `{# uix.debug #}` pour déboguer les modèles.
    - Tous les messages de débogage de la console commencent par `UIX`.

## Existe-t-il une liste des différences entre Card-mod et UI eXtension ?

Oui, consultez le tableau ci-dessous.

<!-- markdownlint-disable MD033 -->
| Fonctionnalité | Card-mod | UIX |
| --- | :---: | :---: |
| Charge correctement les variables de thème `...-yaml` | ❌<br>Depuis 2026.8.0 | Oui |
| Gère correctement la variable de thème `...-more-info(-yaml)` | ❌<br>Depuis 2026.3.0 | Oui |
| Adapte correctement les dialogues pour la variable de thème `...-dialog(-yaml)` | ❌<br>Depuis 2026.3.0 | Oui |
| [Outils d'inspection du DOM](../concepts/dom.md#dom-inspection-helpers) | Non | Oui |
| [Sélection de chemin hôte/élément](../concepts/dom.md#hostelement-path-selection) | Non | Oui |
| [Sélecteur de recherche rapide](../concepts/dom.md#express-search-selector) | Non | Oui |
| [Forge](../forge/index.md) (élément Lovelace personnalisé) | Non | Oui |
| [Fonderies](../forge/foundries.md) (configurations Forge réutilisables) | Non | Oui |
| [Macros](../using/templates.md#macros) (modèles Jinja réutilisables) | Non | Oui |
| [Sparks](../forge/sparks/index.md) (comportements autonomes ajoutés aux éléments Forge) | Non | Oui |
| [Broker](../broker/index.md) (création d'interactions déclaratives avec les événements du frontend) | Non | Oui |
| [Limitation des mises à jour d'état du frontend](../extras/frontend-states-throttling.md) (facultative) | Non | Oui |
| [Délai d'application des styles aux dialogues](../extras/dialog-styling-delay.md) (facultatif) | Non | Oui |
| [Arrière-plans des vues du tableau de bord](../using/view-backgrounds.md) | Non | Oui |
| [Arrière-plans des sections](../using/section-backgrounds.md) | Non | Oui |
| [Arrière-plans des vues](../using/view-backgrounds.md) | Non | Oui |
| [Style des icônes — remplacement par entité](../using/icons.md#specifying-for-an-entity-override) | Non | Oui |
| [Style des images d'entité](../using/images.md) | Non | Oui |
| [Style des panneaux personnalisés](../using/custom-panels.md), y compris ceux chargés dans des iFrames | Non | Oui |
| [Style des panneaux d'applications et d'ingress](https://uix.lf.technology/using/apps/), y compris ceux chargés dans des iFrames | Non | Oui |
| Fenêtre contextuelle pour recharger ou vider le cache | Non | Oui |
| Documentation détaillée avec exemples visuels | Limitée | Oui |
| Mod-card | Oui | Oui |
| Styles CSS dans les thèmes | Oui | Oui |
| Service/action de rechargement ou d'effacement du cache | Oui | Oui |
| Fournit des variables (par exemple l'utilisateur actuel) | Oui | Oui |
| Styles CSS | Oui | Oui |
| URL de ressource | Oui | S.O. |

## UI eXtension rencontre-t-il des problèmes d'URL de ressource ?

Non. En tant qu'intégration, UI eXtension gère directement ses URL de ressources. Aucune intervention n'est nécessaire pour garantir son bon fonctionnement. UI eXtension ajoute dynamiquement sa ressource frontend `uix.js` comme module supplémentaire et ajoute également une ressource de tableau de bord si vous utilisez CAST. À chaque chargement de l'intégration, UI eXtension ajoute automatiquement sa version à ces ressources.

## Faut-il vider manuellement le cache des navigateurs et des applications mobiles après une mise à jour de UI eXtension ?

Non. Lorsque UI eXtension détecte qu'un rechargement est nécessaire pour vider les caches, un message temporaire s'affiche avec le bouton `Reload Now`. La page se recharge automatiquement après 60 secondes.

!!! note
    Le code de rechargement automatique est inclus depuis la version 8.1.0, mais il ne sera disponible qu'après votre prochaine mise à jour. Lors de l'installation de la version 8.1.0, l'appareil utilise encore le code de la version 8.0.1, qui ne comporte pas cette fonction.

## Comment désinstaller UI eXtension ?

La désinstallation de UI eXtension se déroule en deux étapes. Supprimez d'abord son entrée de service dans **Appareils et services**. Désinstallez ensuite l'intégration avec HACS ou, si vous l'avez installée manuellement, en supprimant le dossier `uix` du répertoire `custom_components`.
