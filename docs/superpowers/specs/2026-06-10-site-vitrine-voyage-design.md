# Site vitrine « KAZAFK — Le Voyage » — Document de design

**Date :** 2026-06-10
**Statut :** validé en brainstorming, en attente de relecture finale
**Dépôt cible :** `Kazafk/Kazafk.github.io` → publié sur https://kazafk.github.io

## Objectif

Un site vitrine one-page immersif présentant les projets GitHub de Kazafk. Chaque projet
disposant d'une GitHub Page devient une « étape » d'un voyage vertical ; un clic sur
l'étape ouvre la page du projet. Le site sert de hub : il débranche vers les autres
GitHub Pages du compte.

## Direction artistique (validée)

Inspirée de **Nomadic Tribe** (2019.makemepulse.com, Awwwards SOTD) : récit vertical
pictural, on descend du ciel doré vers les abysses. La base visuelle est le design
généré par **Google Stitch** (projet Stitch `14451318137269571978`, copie de référence
dans `docs/design-reference/stitch-design.html`).

- **Design system « KAZAFK Journey »** : mode sombre, fond `#001714`, accent or `#ecd06f`,
  turquoise `#49c5b6`, brun terre `#987654` ; titres **Playfair Display**, textes
  **Hanken Grotesk** (Google Fonts).
- **Illustrations** : 3 scènes picturales générées par Stitch, stockées dans le dépôt
  (`assets/img/scene-ciel.jpg`, `scene-profondeurs.jpg`, `scene-abysses.jpg`,
  1376×768 natif). Un **voile de grain** CSS les recouvre pour masquer la douceur
  d'upscale sur grands écrans et renforcer le rendu pictural.
- **Langue :** français.

## Structure de la page (validée)

1. **Hero « ciel »** — titre KAZAFK massif (Playfair Display), sous-titre
   « Un voyage à travers mes projets », oiseaux animés, invitation à descendre.
2. **Une étape par projet** — sections plein écran alternées sur fond illustré :
   numéro d'étape, nom du projet, description, badge langage, bouton
   « Visiter le projet → » vers `https://kazafk.github.io/<repo>/`.
   Le hero utilise `scene-ciel.jpg` ; les étapes alternent `scene-profondeurs.jpg`
   et `scene-abysses.jpg` en boucle, quelle que soit la quantité de projets.
3. **Footer « Fin du voyage »** — lien vers le profil GitHub, bouton
   « Remonter à la surface » (scroll vers le haut).

## Expérience (validée)

- **Animations GSAP ScrollTrigger** (CDN) : parallaxe sur les fonds, apparition des
  cartes, transitions de couleur entre zones, indicateur de progression discret.
- **Faune animée** (canvas léger) : oiseaux dans le ciel, poissons dans les
  profondeurs — mouvement ambiant indépendant du scroll.
- **Curseur personnalisé** : lueur suivant la souris, désactivé sur écran tactile.
- Pas de narration textuelle en chemin, pas d'ambiance sonore (choix explicites).

## Architecture technique (validée)

Site statique sans build, déployé tel quel sur GitHub Pages :

```
index.html          – structure + contenu fallback statique
css/styles.css      – tokens du design system, styles, grain, reduced-motion
js/main.js          – orchestration (ordre : données → DOM → animations)
js/github.js        – récupération des projets (API GitHub)
js/journey.js       – animations GSAP ScrollTrigger
js/fauna.js         – canvas oiseaux/poissons
js/cursor.js        – curseur lumineux
assets/img/         – 3 scènes Stitch + grain
```

Modules indépendants communiquant par le DOM : `github.js` génère les sections
d'étapes, puis `journey.js` les anime. GSAP + ScrollTrigger via CDN.

## Données

- `GET https://api.github.com/users/Kazafk/repos?per_page=100`
- Filtre : `has_pages === true`, exclusion de `Kazafk.github.io` lui-même.
- Tri : date de mise à jour (`pushed_at`) décroissante.
- Champs utilisés : `name`, `description`, `language`, lien construit
  `https://kazafk.github.io/<name>/`.
- Cache `localStorage` 1 h (limite API : 60 req/h/IP, largement suffisant).

## Robustesse et accessibilité

- **Fallback statique** : un instantané JSON des 3 projets actuels est embarqué dans
  `index.html`. Si l'API échoue (panne, rate-limit), le site affiche cet instantané —
  jamais de page vide.
- **`prefers-reduced-motion`** : parallaxe, faune et curseur désactivés, contenu
  directement visible.
- **Mobile** : scroll natif vertical, curseur custom désactivé, densité de faune
  réduite, typo fluide (`clamp()`).
- **Sans JavaScript** : hero, instantané des projets et lien GitHub restent visibles.
- Contrastes conformes (or sur vert sombre), liens au clavier, `alt`/`aria-label`
  sur les éléments interactifs.

## Vérification et déploiement

- Test local : serveur HTTP simple (`python -m http.server`), vérification visuelle
  desktop + simulation mobile, passe Lighthouse (performance ≥ 90, accessibilité ≥ 90).
- Déploiement : création du dépôt distant `Kazafk/Kazafk.github.io` (API GitHub,
  identifiants du Credential Manager), push `main`, activation de GitHub Pages
  (branche `main`, racine). Site en ligne sur https://kazafk.github.io.

## Hors périmètre (YAGNI)

- Multilingue, ambiance sonore, narration au scroll, WebGL/three.js, framework/build,
  analytics, formulaire de contact.
