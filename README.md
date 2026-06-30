# Filtres exposés DSFR — bac à sable

Prototype interactif pour **rationaliser les filtres exposés des vues Drupal**, aligné sur
le [Système de Design de l'État Français (DSFR)](https://www.systeme-de-design.gouv.fr/).

Modèle de référence : [choisirleservicepublic.gouv.fr](https://choisirleservicepublic.gouv.fr/),
transposé sur une vue d'[economie.gouv.fr](https://www.economie.gouv.fr/).

C'est un **document de specs vivant** : il montre comment chaque type de filtre
(texte, booléen, monovalué, multivalué, date) se rend dans chacun des trois emplacements,
et **explicite la règle de mise en page appliquée** pour que l'intégration Drupal soit
sans ambiguïté.

## Deux rendus, un même bac à sable

Le même prototype existe en deux versions, **pilotées par le même disclose de paramétrage**
et reliées par un **bouton de bascule en haut à droite** :

| Page | Rendu de la zone « Affiner votre recherche » |
|------|----------------------------------------------|
| `index.html` | **Prototype** — HTML/CSS/JS natif stylé « à la main » sur les jetons DSFR. |
| `index-dsfr.html` | **100 % DSFR natif** — uniquement des composants officiels (`fr-input`, `fr-checkbox-group`, `fr-radio-group`, `fr-toggle`, `fr-modal`, `fr-tag`, `fr-btn`…) et des classes utilitaires. |

`index-dsfr.html` sert de **preuve de conformité** : il vérifie que le rendu prototypé est
bien reproductible avec le DSFR officiel. Seul le **disclose de paramétrage** y reste hors
DSFR (volontairement). Les filtres *primaires* y ouvrent une **modale DSFR** (`fr-modal`) au
lieu du panneau latéral du prototype, le DSFR n'ayant pas de composant « panneau ».

## Lancer

Aucune dépendance, aucun build. Ouvrir `index.html` (ou `index-dsfr.html`) dans un
navigateur, ou servir le dossier :

```bash
python3 -m http.server 8000
# puis http://localhost:8000/        (prototype)
#   ou http://localhost:8000/index-dsfr.html   (100 % DSFR)
```

Tout est **embarqué dans `assets/`** : police Marianne, jetons de couleur DSFR, et le DSFR
officiel vendoré (`assets/dsfr/`, v1.14.4 — CSS, JS, icônes). Le prototype `index.html`
charge en plus les icônes [Remix Icon](https://remixicon.com/) depuis un CDN.

## Comment ça marche

Une **zone de réglages** pilote un **canvas de rendu** :

- **Type d'élément** — Texte · Booléen · Monovalué · Multivalué · Date
- **Emplacement** — Primaire · Secondaire · Tertiaire
- **Longueur des libellés** (Courts / Longs) et **Nombre d'options** (2–14) pour les types à choix

Un bandeau **« Règle appliquée »** affiche en direct la décision de mise en page. La sélection
est **partagée entre les trois zones** : cocher une valeur en Secondaire met à jour le compteur
de la Primaire et les tags de la Tertiaire.

## Les trois zones

| Zone | Rôle | Rendu |
|------|------|-------|
| **Primaire** | Filtre **replié dans un bouton** qui ouvrirait un panneau/modale | `Libellé ›` + compteur (multivalué) ou nom de la valeur (monovalué). Texte et booléen sont exposés directement : pas de bouton. |
| **Secondaire** | Filtre **déplié en clair**, sans panneau | L'input lui-même : cases à cocher, radios, champ date/texte, interrupteur. C'est ici que se jouent les décisions de layout. |
| **Tertiaire** | **Récapitulatif** des valeurs choisies | Tags retirables. Alternative pour afficher les sélections faites en Primaire. |

## Règles de mise en page (le cœur des specs)

### Zone secondaire — deux décisions

**1. Orientation du groupe** (label + valeurs en ligne, ou label au-dessus en bloc) :

```
groupeLigne = (type == booléen)
           || (type à choix && nbOptions <= 3 && libellés courts)
```

Sinon le groupe est en **bloc** (label sur sa propre ligne, valeurs dessous).

**2. Disposition des valeurs** (cases/radios) :

```
valeursVerticales = (type à choix) && (libellés longs || nbOptions > 10)
```

- **liste verticale** quand les libellés sont longs ou les options nombreuses (> 10) ;
- sinon **inline-wrap** : les valeurs s'alignent et passent naturellement à la ligne
  (conteneur `flex; flex-wrap:wrap`, jamais de scroll horizontal).

> Réponse directe à la question « un multivalué à 7 valeurs courtes ? » → **inline-wrap**,
> cases sur une ligne qui passent à la ligne au besoin.

Cas particuliers fixes :
- **Texte** → groupe en bloc, champ pleine largeur.
- **Date** → groupe en bloc, deux champs `Du` / `Au` en ligne.

### Zone tertiaire — bloc vs inline

Le conteneur de tags est toujours `flex; flex-wrap:wrap; align-items:baseline`. Le label
bascule en `flex-basis:100%` (valeurs **en bloc dessous**) uniquement quand :

```
tertiaireBloc = (type == multivalué) && (nbTags >= 5 || libellés longs)
```

Sinon label + tags restent **inline** et wrappent naturellement. Le booléen n'affiche pas
de label : son nom seul sert de tag.

## Transposition Drupal

Le prototype est volontairement **sans framework** (HTML/CSS/JS natif) pour servir de
référence d'intégration. Côté Drupal, chaque filtre exposé d'une vue correspond à un type
ci-dessus ; les règles `groupeLigne` / `valeursVerticales` / `tertiaireBloc` sont à porter
dans le template (Twig) ou le thème du formulaire exposé, en réutilisant les composants et
jetons DSFR (`assets/colors_and_type.css`).

## Structure

```
index.html                  Bac à sable — rendu prototype (custom)
index-dsfr.html             Bac à sable — rendu 100 % DSFR natif
assets/colors_and_type.css  Jetons DSFR (couleurs, type) + @font-face Marianne
assets/fonts/               Police Marianne (woff/woff2, tous les poids)
assets/dsfr/                DSFR officiel vendoré v1.14.4 (CSS, JS, icônes, polices)
```

---

Issu d'une maquette [Claude Design](https://claude.ai/design), réimplémentée en code natif.
