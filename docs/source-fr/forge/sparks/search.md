---
description: Recherchez des éléments dans le Shadow DOM et modifiez leurs classes, attributs ou textes avec le spark Search de UIX Forge.
icon: material/magnify
---

# :mag: Spark de recherche

Le spark `search` recherche des éléments dans un conteneur à l'aide d'un sélecteur CSS. Il peut filtrer les résultats à l'aide d'une expression régulière appliquée au texte, puis modifier les classes, les attributs ou le texte de chaque élément correspondant. Il installe également un `MutationObserver` afin de traiter automatiquement les nouveaux éléments, comme les événements d'un calendrier après un changement de mois.

## Utilisation de base

Ajoutez une entrée `search` à `forge.sparks` :

```yaml
type: custom:uix-forge
forge:
  mold: card
  sparks:
    - type: search
      for: hui-calendar-card $ ha-full-calendar $
      query: .fc-event-title
      text: "Meeting"
      actions:
        add_class:
          - highlight
element:
  type: calendar
  entities:
    - calendar.work
```

`query` est un sélecteur CSS transmis à `querySelectorAll` sur le conteneur trouvé. `text` est une expression régulière comparée à tout le texte de chaque élément correspondant, y compris celui de ses enfants tels que `<a>` ou `<span>`. Seuls les éléments qui passent ce filtre reçoivent les `actions`.

## Configuration

| Clé | Type | Obligatoire | Valeur par défaut | Description |
| --- | ---- | -------- | ------- | ----------- |
| `type` | `string` | ✅ | — | Doit être défini sur `search`. |
| `for` | `string` | | `element` | Sélecteur UIX du conteneur à explorer. `$` permet de traverser les racines Shadow DOM (voir [Navigation dans le DOM](../../concepts/dom.md)). Par défaut, `element` désigne la racine de l'élément créé avec UIX Forge. |
| `query` | `string` | ✅ | — | Sélecteur CSS transmis à `querySelectorAll` sur le conteneur trouvé. Les actions configurées sont appliquées à tous les éléments correspondants. |
| `text` | `string` | | — | Expression régulière. Si elle est définie, seuls les éléments dont le texte complet correspond, y compris le texte de leurs enfants, sont traités. |
| `actions` | `object` | | `{}` | Modifications à appliquer à chaque élément correspondant. Voir la section [Actions](#actions). |

!!! tip
    Utilisez le helper de console [`uix_forge_path()`](../../concepts/dom.md#uix_forge_path0-forge-helper) pour trouver le sélecteur exact à utiliser avec `for`.

### Actions

L'objet `actions` peut contenir n'importe quelle combinaison des clés suivantes. Elles sont toutes facultatives.

| Clé | Type | Description |
| --- | ---- | ----------- |
| `add_class` | `list[string]` | Noms des classes CSS à ajouter à chaque élément correspondant. |
| `remove_class` | `list[string]` | Noms des classes CSS à retirer de chaque élément correspondant. |
| `add_attribute` | `list[{attribute, value}]` | Attributs HTML à définir. Chaque entrée doit contenir un nom `attribute` et une chaîne `value`. |
| `remove_attribute` | `list[string]` | Noms des attributs HTML à supprimer de chaque élément correspondant. |
| `replace_text` | `string` \| `{find, replace}` | Remplacement par expression régulière dans chaque nœud texte de l'élément. Une **chaîne** sert de motif et les occurrences sont supprimées. Un **objet** contenant `find` et `replace` remplace chaque occurrence par la valeur de `replace`. |
| `prepend_text` | `string` | Texte à ajouter au début de chaque nœud texte de l'élément. |
| `append_text` | `string` | Texte à ajouter à la fin de chaque nœud texte de l'élément. |

## Exemples

### Ajouter une classe CSS aux événements correspondant à une expression régulière

```yaml
type: custom:uix-forge
forge:
  mold: card
  sparks:
    - type: search
      for: hui-calendar-card $ ha-full-calendar $
      query: .fc-event-title
      text: ^(Future|Timetravel)
      actions:
        add_class:
          - future-event
element:
  type: calendar
  entities:
    - calendar.calendar_1
    - calendar.calendar_2
  uix:
    style:
      ha-full-calendar $: |
        .future-event {
          background: teal;
          color: white;
          font-weight: 900;
        }
        .fc-daygrid-event:has(.future-event) {
          background-color: teal !important;
          border-color: blue !important;
          border-width: 2px;
        }
        .fc-event-time:has(+ .future-event) {
          background: teal;
          color: white;
          font-weight: 900;      
        }
```

![Search spark calendar example](../../assets/page-assets/forge/sparks/search-calendar.png)

### Supprimer un attribut de tous les éléments correspondants

Supprimez l'attribut `title` de chaque lien d'une carte Markdown afin d'empêcher l'affichage de l'infobulle native du navigateur :

```yaml
type: custom:uix-forge
forge:
  mold: card
  sparks:
    - type: search
      for: >-
        hui-entities-card $ div:nth-child(2) hui-sensor-entity-row $ hui-generic-entity-row $
      query: .info
      text: Carbon dioxide
      actions:
        replace_text:
          find: Carbon dioxide
          replace: CO2
    - type: search
      for: hui-entities-card $ div:nth-child(2) hui-sensor-entity-row $
      query: hui-generic-entity-row
      text: ppm
      actions:
        replace_text:
          find: ppm
          replace: parts per million
element:
  type: entities
  entities:
    - sun.sun
    - sensor.carbon_dioxide
```

![Search spark entities example](../../assets/page-assets/forge/sparks/search-entities.png)

### Ajouter du texte au début et à la fin

Ajoutez un préfixe et un suffixe aux informations de la première ligne d'entités :

```yaml
type: custom:uix-forge
forge:
  mold: card
  sparks:
    - type: search
      for: hui-entities-card $ div:nth-child(1) hui-sensor-entity-row $ hui-generic-entity-row $
      query: .info
      actions:
        prepend_text: "[ "
        append_text: " ]"
element:
  type: entities
  entities:
    - sensor.carbon_dioxide_battery
```

![Search prepend append example](../../assets/page-assets/forge/sparks/search-prepend-append.png)

### Rechercher dans le texte des éléments enfants

Le filtre `text` compare le **texte complet** de chaque élément, y compris le texte contenu dans des éléments enfants tels que `<a>` ou `<span>`.

Cet exemple reprend celui du calendrier et met également en évidence les événements de la liste en ajoutant la classe `future-event` aux éléments `.fc-list-event-title`. Il ajoute aussi des styles avec un sélecteur qui cible le frère précédent de `.future-event`, à l'aide de la pseudo-classe `:has` et du combinateur de frères adjacents `+`.

```yaml
type: custom:uix-forge
forge:
  mold: card
  sparks:
    - type: search
      for: hui-calendar-card $ ha-full-calendar $
      query: .fc-event-title
      text: ^(Future|Timetravel)
      actions:
        add_class:
          - future-event
    - type: search
      for: hui-calendar-card $ ha-full-calendar $
      query: .fc-list-event-title
      text: ^(Future|Timetravel) # Correspond même si `Future` se trouve dans le lien enfant <a>
      actions:
        add_class:
          - future-event
element:
  type: calendar
  initial_view: listWeek
  entities:
    - calendar.calendar_1
    - calendar.calendar_2
  uix:
    style:
      ha-full-calendar $: |
        .future-event {
          background: teal;
          color: white;
          font-weight: 900;
        }
        .fc-daygrid-event:has(.future-event) {
          background-color: teal !important;
          border-color: blue !important;
        }
        .fc-event-time:has(+ .future-event) {
          background: teal;
          color: white;
          font-weight: 900;      
        }
        .fc-list-event-graphic:has(+ .future-event) {
          background: teal;
          color: white;
          font-weight: 900;   
          border-bottom-left-radius: 0;
          border-bottom-right-radius: 0;
        }
        .fc-list-event-time:has(~ .future-event) {
          background: teal;
          color: white;
          font-weight: 900;   
          border-bottom-left-radius: 0;
          border-bottom-right-radius: 0;
        }
        .fc-list-event-title.future-event {
          border-bottom-left-radius: 0;
          border-bottom-right-radius: 0;
        }
```

![Search calendar example 2](../../assets/page-assets/forge/sparks/search-calendar-2.png)

![Search calendar example 3](../../assets/page-assets/forge/sparks/search-calendar-3.png)

!!! note
    - Les actions s'appliquent à **tous** les éléments renvoyés par `query`. Utilisez `text` pour limiter la sélection à ceux dont le texte correspond à une expression régulière.
    - Le spark observe le conteneur avec un `MutationObserver` et traite automatiquement les éléments ajoutés dynamiquement, par exemple après un changement de mois dans un calendrier.
