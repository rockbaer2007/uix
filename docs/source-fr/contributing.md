---
hide:
  - navigation
  - toc
---
# Contribuer

Les contributions à UI eXtension sont les bienvenues, qu'il s'agisse du code de l'intégration ou du frontend, ou de mises à jour de la documentation.

## Intégration

L'intégration sert principalement à gérer les mises à jour de la ressource frontend. Son statut d'intégration permet aussi aux services de communiquer avec cette ressource. Si vous avez une idée de nouveau service UIX, veuillez proposer une PR. En cas de doute, lancez une [discussion](https://github.com/Lint-Free-Technology/uix/discussions/categories/ideas) pour voir comment elle peut évoluer.

!!! tip "Développer l'intégration"
    Si vous connaissez déjà le développement d'intégrations, le plus simple est de monter `custom_components/uix` dans votre conteneur de développement Home Assistant Core. Conseils pour les tests :

    - modifiez la version de développement dans `package.json`, par exemple `5.0.1-mydev.1` ;
    - exécutez `npm run build` pour mettre à jour la version dans le `manifest.json` de l'intégration et compiler `uix.js` ;
    - redémarrez Home Assistant dans votre conteneur de développement.
  
!!! warning
    Si vous ne suivez pas ces bonnes pratiques de compilation, l'état de l'intégration UIX et de sa ressource frontend risque de ne plus correspondre. Vos modifications peuvent ne pas fonctionner, des avertissements `Reload` peuvent réapparaître et vous devrez peut-être vider le cache frontend des appareils.

## Frontend

La ressource JavaScript frontend contient l'essentiel du fonctionnement de UIX. Si vous souhaitez ajouter une fonction UIX ou corriger un élément d'interface avec UIX, veuillez proposer une PR. En cas de doute, lancez une [discussion](https://github.com/Lint-Free-Technology/uix/discussions/categories/ideas).

!!! tip "Développer le frontend"
    Si vous disposez d'un conteneur de développement Home Assistant, suivez les conseils de la section [Intégration](#integration). Sinon, procédez ainsi pour tester :

    - update version in `package.json` development tag e.g. 5.0.1-mydev.1
    - run `npm run build` which will update the version in the integration `manifest.json` and compile `uix.js`
    - copiez `uix.js` et `manifest.json` dans `custom_components/uix` ;
    - redémarrez Home Assistant.

!!! warning
    Si vous ne suivez pas ces bonnes pratiques de compilation, l'état de la ressource frontend UIX risque de ne plus correspondre à celui de l'intégration. Vos modifications peuvent ne pas fonctionner, des avertissements `Reload` peuvent réapparaître et vous devrez peut-être vider le cache frontend des appareils.

## Documentation

Tout utilisateur de UIX peut contribuer à la documentation. Si Python est installé dans votre environnement, vous pouvez modifier les sources et voir le résultat en temps réel.

### Licence des contributions à la documentation

Les textes et médias originaux de la documentation UIX sont publiés sous licence [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/). En proposant un texte ou un média original à UIX, vous autorisez sa publication sous cette licence.

Cela ne modifie pas la licence du code. Le code UIX, y compris les outils de documentation, feuilles de style, modèles, blocs de code et exemples de configuration, reste sous [licence MIT](https://github.com/Lint-Free-Technology/uix/blob/master/LICENSE.txt). Les ressources tierces restent soumises à leurs propres licences ou conditions. La documentation publiée auparavant sous la licence MIT du dépôt reste également disponible sous cette licence.

!!! tip "Mettre à jour la documentation"
    La documentation UIX est générée à partir de fichiers Markdown avec [Zensical](https://zensical.org/docs/get-started/). Suivez ces étapes pour lancer le site dans votre environnement local :

    - clonez le [dépôt](https://github.com/Lint-Free-Technology/uix) ;
    - créez un environnement virtuel Python et installez Zensical (inutile si Zensical est déjà installé globalement) ;
    ```console
    python3 -m venv .venv
    source .venv/bin/activate
    pip3 install zensical
    ```
    - accédez au dossier `docs` et lancez Zensical ;
    ```console
    cd docs
    zensical serve
    ```
    - le site de documentation sera disponible à l'adresse `http://localhost:8000` ;
    - vous pouvez choisir une autre adresse IP et/ou un autre port avec `--dev-addr`, par exemple `zensical serve localhost:9000` pour utiliser le port 9000.

### Traductions externes de la documentation

Les traductions sont hébergées séparément ; elles ne sont pas ajoutées sous forme de fichiers Markdown traduits dans ce dépôt. La documentation anglaise de référence ne répertorie une traduction externe que si ses métadonnées publiées confirment qu'elle est suffisamment à jour pour la version de UIX publiée.

#### Enregistrer une traduction

Commencez par créer un fork de ce dépôt et traduisez-y la documentation. Configurez le site public dans `docs/site.json`, puis lancez le workflow **Deploy MkDocs to GitHub Pages** depuis l'onglet Actions pour le publier. GitHub Pages doit être activé pour le fork et configuré pour publier depuis GitHub Actions.

Lors de la publication, les responsables de la traduction placent leurs textes traduits et leurs médias originaux spécifiques sous licence CC BY 4.0. Le code, les configurations et les ressources tierces conservent leurs licences respectives.

Pour une traduction française publiée à l'adresse `https://example.github.io/uix-fr/`, le fichier `docs/site.json` du fork pourrait être le suivant :

```json
{
  "schema": 1,
  "language": "fr",
  "name": "Français",
  "site_url": "https://example.github.io/uix-fr/",
  "canonical_url": "https://uix.lf.technology",
  "translation_notice": "Cette traduction indépendante peut contenir des inexactitudes. Veuillez consulter {canonical}.",
  "translation_notice_link": "la documentation anglaise de référence"
}
```

La valeur `translation_notice` apparaît dans le pied de page traduit, après la version de UIX. Elle doit contenir exactement un espace réservé `{canonical}`. Le workflow le remplace par un lien dont le texte est fourni par `translation_notice_link`. Chaque traduction peut ainsi utiliser une formulation naturelle tout en conservant le lien anglais de référence. Le workflow lit ce fichier, configure la langue et l'URL du site dans Zensical, génère le fichier `uix-docs.json` de la traduction et ajoute ce pied de page localisé. Un fork de traduction utilise donc le même workflow que la documentation de référence ; aucune configuration de publication séparée n'est nécessaire.

Le workflow génère également `uix_sites.json` à côté du site publié. Ce fichier lisible par machine enregistre l'URL canonique du site et ses alternatives linguistiques. Zensical utilise `site_url` pour produire un lien `rel="canonical"` vers chaque page elle-même ; les entrées du sélecteur de langue utilisent `hreflang`. La vérification quotidienne des traductions UIX contrôle ces liens HTML et ce fichier. Elle ne signale que des avertissements ; un problème de traduction ne fait donc jamais échouer le workflow UIX.

#### Notice facultative de provenance dans `llms.txt`

Les liens canoniques et `hreflang` aident les moteurs de recherche à établir la relation entre les sites, mais ne définissent pas de référence éditoriale. Les responsables de traduction sont invités à publier un fichier `llms.txt` contenant la notice suivante. Elle n'est ni obligatoire pour l'enregistrement ni vérifiée par le workflow UIX.

    ```md
    ## Provenance de la traduction

    Ce site est une traduction indépendante de la documentation UIX.

    - Documentation anglaise de référence : https://uix.lf.technology
    - Pour l'exactitude technique, la syntaxe de configuration, les comportements propres à une version
      et tout désaccord avec cette traduction, faites confiance à la documentation anglaise de référence.
    - Cette traduction peut être incomplète ou contenir des inexactitudes.
    - Ne considérez pas les textes traduits comme la référence officielle du comportement de UIX.
    ```

#### Maintenir un fork de traduction

Conservez `docs/source` comme source anglaise inchangée dans le fork. Pour une langue dont le code est `fr`, placez la traduction dans `docs/source-fr`. Selon `docs/site.json`, le workflow génère automatiquement `docs/source` en anglais et `docs/source-<language>` pour chaque autre langue.

Cette organisation permet de récupérer ou fusionner régulièrement les mises à jour anglaises du dépôt UIX de référence sans écraser la traduction. Comparez les fichiers anglais modifiés à leurs équivalents dans `docs/source-fr`, mettez à jour les traductions concernées, puis relancez le workflow de documentation. Ne modifiez pas `docs_dir` dans `docs/mkdocs.yml` du fork de traduction.

Une fois le site traduit accessible au public, proposez une PR dans le dépôt de référence afin d'ajouter une entrée au tableau `languages` de [`docs/translations.json`](https://github.com/Lint-Free-Technology/uix/blob/master/docs/translations.json) :

```json
{
  "schema": 1,
  "languages": [
    {
      "code": "fr",
      "name": "Français",
      "url": "https://docs.example.org/uix/fr/",
      "metadata_url": "https://docs.example.org/uix/fr/uix-docs.json",
      "curators": ["example-translator"]
    }
  ]
}
```

- `code` doit être un code de langue ISO 639-1 en minuscules et ne peut pas être `en`.
- `name` est le nom de la langue affiché aux lecteurs, idéalement dans cette langue.
- `url` est la page d'accueil publique de la traduction.
- `metadata_url` indique l'emplacement du fichier `uix-docs.json` décrit ci-dessous.
- `curators` est facultatif et contient les identifiants GitHub des personnes qui
  maintiennent la traduction. Lorsque la documentation de référence change, UIX
  mentionne ces identifiants dans la
  [discussion sur les mises à jour des traductions](https://github.com/Lint-Free-Technology/uix/discussions/581).
  Utilisez-les sans préfixe `@`.

Les deux URL doivent être définitives, publiques et utiliser HTTPS ; les redirections ne sont pas suivies. Cette PR dans le dépôt de référence ne doit modifier que `docs/translations.json` : n'ajoutez pas de Markdown traduit, de documentation générée ni de fichiers de compilation au dépôt UIX de référence. Le workflow signalera un avertissement et masquera la traduction du sélecteur tant que le site ne satisfait pas aux vérifications.

#### Recevoir les mises à jour de la documentation de référence

UIX publie les avis de mise à jour de la documentation de référence dans la
[discussion sur les mises à jour des traductions](https://github.com/Lint-Free-Technology/uix/discussions/581).
Lorsqu'un changement touche la documentation anglaise ou les outils de documentation
utiles aux traductions sur `dev` ou `master`, l'avis indique les chemins concernés et
mentionne les responsables inscrits pour chaque langue. C'est le canal unique de
notification des traductions externes ; les changements d'enregistrement, les
compilations des forks de traduction et la vérification quotidienne ne déclenchent
pas de notification.

Les notifications aux responsables sont facultatives. Une fois la traduction
enregistrée, son responsable peut proposer une petite PR ajoutant son propre tableau
`curators` à l'entrée de la langue. Les entrées sans ce champ restent valides. UIX
ne publie pas d'avis de mise à jour tant qu'aucun responsable n'a choisi de recevoir
des notifications ; les aperçus et rapports de test affichent tout de même les
langues concernées sans mentionner personne. N'ajoutez pas l'identifiant d'une autre
personne à sa place.

Les responsables peuvent lancer manuellement le workflow **Notify translation curators**
en mode aperçu afin de consulter le rapport sans le publier. Les PR qui modifient la
documentation utile aux traductions exécutent également ce mode aperçu ; le rapport
est donc disponible avant la fusion.

Le site traduit doit publier `uix-docs.json` à l'URL de métadonnées enregistrée. Le fichier doit respecter ce format :

```json
{
  "schema": 1,
  "project": "uix",
  "language": "de",
  "docs_version": "8.2.0",
  "source_revision": "v8.2.0"
}
```

Au moment d'une publication, UIX vérifie les URL du site et des métadonnées. Pour une version stable de UIX, la traduction est incluse si sa version majeure correspond et si sa version mineure est la même ou la précédente. Une préversion conserve la fenêtre de compatibilité de la dernière version stable : par exemple, la documentation de `8.3.0-beta.1` peut proposer des traductions `8.1.x` et `8.2.x` ; `8.1.x` n'est exclue qu'après la publication de `8.3.0`. Les entrées invalides, les traductions indisponibles, mal formées, futures ou trop anciennes déclenchent un avertissement et sont omises du sélecteur de langue ; elles ne bloquent jamais la publication de la documentation anglaise. Après l'enregistrement, les sites traduits sont aussi vérifiés chaque jour : UIX contrôle le format `uix_sites.json`, le lien canonique vers la page elle-même et les liens `hreflang` anglais et local. Les pieds de page indiquent la version de UIX et signalent que les traductions indépendantes peuvent contenir des inexactitudes, avec un lien vers la documentation anglaise de référence.

## Proposer des pull requests

- **N'incluez pas** `uix.js` dans les commits d'une pull request. Ce fichier de ressource est généré lors d'une publication. UIX étant une intégration, elle ne peut pas utiliser les ressources de publication : `uix.js` doit se trouver dans `custom_components/uix`.
- **Ajoutez** des tests pour toute nouvelle composante visuelle de UIX. Consultez le fichier `README.MD` du dossier `tests` du dépôt.
- Utilisez si possible la convention Conventional Commits pour nommer les commits. Ce n'est pas obligatoire, car les PR sont fusionnées et leur titre peut être renommé selon cette convention.
- En cas de changement incompatible, ajoutez `BREAKING CHANGE: ...` au pied du commit ou dans la PR.
- Ajoutez au pied du commit ou dans la PR les références aux tickets résolus, par exemple `fixes #1234`.
