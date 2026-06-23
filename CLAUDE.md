# CLAUDE.md — Portfolio Kazafk

Contexte projet pour Claude Code. Lis ce fichier avant toute modification.

## Vue d'ensemble

Portfolio personnel mono-page, hébergé sur **GitHub Pages** à l'adresse
`https://kazafk.github.io`. Site **100 % statique, sans build** : un seul
fichier `index.html` contient tout le HTML, le CSS et le JS (inline).

Thème visuel actuel : **« maps, sand & stones »** — esthétique cartographique,
palette sable / terre cuite / pierre, fond de courbes de niveau animées.
(L'ancien thème était un terminal IBM 3270 vert phosphore ; il a été remplacé.)

## Structure du dépôt

```
/
├── index.html      # tout le site (HTML + CSS + JS inline)
├── README.md       # présentation publique du dépôt
└── CLAUDE.md       # ce fichier
```

Pas de `package.json`, pas de bundler, pas de framework. Dépendances chargées
par CDN dans `<head>` : GSAP + ScrollTrigger (`cdn.jsdelivr.net`) et les
polices Google (`Fraunces`, `JetBrains Mono`).

## Anatomie de `index.html`

Le `<body>` a 3 sections : `#hero`, `#projects`, `#footer`.
Le `<script>` inline final contient 3 modules indépendants (IIFE / sections) :

1. **`(function topo())`** — fond topographique animé sur `<canvas id="topo-canvas">`.
   - Champ scalaire = somme de gaussiennes mobiles (`PEAKS`) + ondulation sinus.
   - `fieldValue(nx, ny)` : valeur continue du champ (source unique de vérité).
   - `computeField(t)` : remplit la grille via `fieldValue`.
   - `drawContours(iso, index)` : **marching squares**, 1 passe par iso-niveau ;
     courbe maîtresse (plus épaisse) tous les 4 niveaux.
   - `drawBenchmarks()` : repères « + » sur les sommets ; l'altitude affichée est
     `fieldToAlt(fieldValue(...))` → **toujours cohérente avec les courbes**.
   - Échelle : `ISO_STEP = 0.12` (unité de champ) ↔ 50 m → `M_PER_UNIT ≈ 416,7`.
   - ~30 fps, pause auto quand l'onglet est masqué (`visibilitychange`).

2. **`(function summits())`** — défilé des sommets de France dans l'en-tête.
   - Tableau `S` : `[nom, latitude, longitude, altitude_m]`, **56 entrées**.
   - Source : Wikipédia « Liste des sommets de France les plus proéminents »
     (proéminence ≥ 1000 m). Coordonnées décimales, altitudes en mètres.
   - `fmtCoord(lat, lon)` formate en `DD.DD°N · DD.DD°E` (O = Ouest, S = Sud).
   - Rotation toutes les 3,8 s avec fondu (opacité sur `#summit-cycle`).

3. **Section API GitHub** — cartes projets dynamiques.
   - `fetchRepos()` : `GET api.github.com/users/Kazafk/repos`, filtre les forks,
     trie par étoiles puis par date de mise à jour. **Non authentifié**
     (limite ~60 req/h par IP — suffisant pour une page perso).
   - `renderCards(repos)` : génère les « parcelles » (tuiles de relevé).
   - `REPO_META` : descriptions FR personnalisées par repo (override de la
     description GitHub). Ajouter une entrée ici pour soigner un projet.
   - `attachTilt()` : effet d'inclinaison 3D au survol (souris) + gyroscope (mobile).
   - `escapeHTML()` + `safeUrl()` : toujours les utiliser pour toute donnée API
     injectée dans le DOM (protection XSS / liens). **Ne pas régresser là-dessus.**

## Design system (tokens dans `:root`)

| Token            | Hex       | Usage                          |
|------------------|-----------|--------------------------------|
| `--paper`        | `#E7CDA6` | fond sable clair               |
| `--paper-deep`   | `#DBB988` | fond sable dense               |
| `--sand-line`    | `#C79A6A` | bordures, lignes topo claires  |
| `--terracotta`   | `#B5532A` | accent principal               |
| `--terra-soft`   | `#C2603A` | accent secondaire              |
| `--rust`         | `#8E3B1C` | terre brûlée, courbes maîtresses |
| `--stone`        | `#5A3A23` | texte secondaire               |
| `--ink`          | `#3A2415` | texte principal, boutons pleins |
| `--glacier`      | `#6F8C86` | accent froid (réservé/eau)     |

Typographie : **Fraunces** (serif éditorial) pour titres/corps, **JetBrains Mono**
pour tout l'habillage technique (coordonnées, stats, labels, échelle).

## Conventions

- Tout reste **dans un seul fichier** (`index.html`). Ne pas éclater en
  fichiers séparés sans demande explicite.
- Langue de l'interface : **français**.
- Pas d'`localStorage`/`sessionStorage` requis ; rester sans dépendance lourde.
- Garder l'accessibilité : contrastes suffisants, `rel="noopener noreferrer"`
  sur les liens externes, `prefers-reduced-motion` à considérer si on enrichit
  les animations.
- Échapper systématiquement les données externes (voir `escapeHTML`/`safeUrl`).

## Aperçu local

Aucune compilation. Pour servir en local :

```bash
python3 -m http.server 8000
# puis ouvrir http://localhost:8000
```

(ou simplement ouvrir `index.html` dans un navigateur — l'API GitHub et les CDN
nécessitent une connexion réseau.)

## Déploiement

GitHub Pages sert la branche `master` à la racine. Tout push sur `master` met le
site à jour automatiquement.

```bash
git add index.html README.md CLAUDE.md
git commit -m "Refonte thème maps/sand/stones + sommets de France"
git push origin master
```

## Pistes / TODO possibles

- [ ] Respecter `prefers-reduced-motion` (figer le canvas + le défilé sommets).
- [ ] Cache léger de l'API GitHub (ex. `sessionStorage`, TTL court) pour éviter
      la limite de débit en cas de rechargements répétés.
- [ ] Bouton thème clair « parchemin » / sombre « nuit désert » (variantes de palette).
- [ ] Réintroduire en option les filtres par langage (retirés volontairement).
- [ ] `<meta>` Open Graph + image de prévisualisation dédiée.
- [ ] Mini rose des vents dans un coin du cadre du hero.

## Notes de données

- Sommets : 56 entrées issues de Wikipédia (proéminence ≥ 1000 m). Si la liste
  évolue, mettre à jour le tableau `S` dans le module `summits` — format
  `[nom, lat, lon, alt_m]`, coordonnées décimales.
- Le Mont Blanc est listé à **4806 m** (valeur Wikipédia de la page source).
