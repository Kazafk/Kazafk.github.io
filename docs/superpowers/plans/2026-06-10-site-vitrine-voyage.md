# Site vitrine « KAZAFK — Le Voyage » — Plan d'implémentation

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Site one-page immersif sur https://kazafk.github.io présentant les GitHub Pages du compte Kazafk comme les étapes d'un voyage vertical (ciel doré → abysses).

**Architecture:** Site statique sans build (HTML + CSS + modules ES vanilla). `github.js` fournit les données (API GitHub avec cache et fallback statique embarqué dans le HTML), `main.js` orchestre, `journey.js` (GSAP ScrollTrigger via CDN) anime, `fauna.js` et `cursor.js` ajoutent l'ambiance. La logique pure est testée avec `node --test` (Node 24, natif).

**Tech Stack:** HTML5, CSS custom (design system « KAZAFK Journey » : Playfair Display + Hanken Grotesk, `#001714`/`#ecd06f`/`#49c5b6`), GSAP 3.12 + ScrollTrigger (CDN jsdelivr), API GitHub REST, `node:test`, GitHub Pages.

**Spec :** `docs/superpowers/specs/2026-06-10-site-vitrine-voyage-design.md`
**Dépôt :** `C:\Repos\Kazafk.github.io` (déjà initialisé, assets dans `assets/img/`)

---

## Structure des fichiers

| Fichier | Responsabilité |
|---|---|
| `index.html` | Structure, fonts, instantané statique des 3 projets (fallback sans JS/API) |
| `css/styles.css` | Tokens du design system, layout, grain, responsive, reduced-motion |
| `js/github.js` | Données : sélection/tri des dépôts, cache localStorage, génération HTML des étapes (fonctions pures testables) |
| `js/main.js` | Orchestration : données → DOM → animations |
| `js/journey.js` | GSAP ScrollTrigger (parallaxe, reveals), indicateur de progression, bouton « remonter » |
| `js/fauna.js` | Canvas : oiseaux (ciel) et poissons (profondeurs) |
| `js/cursor.js` | Curseur lumineux (désactivé tactile / reduced-motion) |
| `tests/github.test.mjs` | Tests `node --test` de toute la logique pure |

Convention : code et identifiants en **français** (cohérent avec le contenu du site), échappement HTML systématique des données venant de l'API.

---

### Task 1 : Fondation HTML + CSS (site statique complet sans JavaScript)

**Files:**
- Create: `index.html`
- Create: `css/styles.css`

- [ ] **Step 1 : Écrire `index.html`**

L'instantané statique reflète l'état réel de l'API au 2026-06-10, trié par `pushed_at` décroissant : sca-water-map (JavaScript), animated-octo-happiness (TypeScript), mmia4 (HTML, « Informations MM »). Alternance des scènes : étape impaire = profondeurs, étape paire = abysses.

```html
<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>KAZAFK — Le Voyage</title>
<meta name="description" content="Un voyage immersif à travers les projets de Kazafk.">
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Playfair+Display:wght@700;900&family=Hanken+Grotesk:wght@400;600&display=swap" rel="stylesheet">
<link rel="stylesheet" href="css/styles.css">
</head>
<body>
<div class="curseur" id="curseur" aria-hidden="true"></div>
<canvas id="faune" aria-hidden="true"></canvas>
<nav class="progression" id="progression" aria-label="Progression du voyage"></nav>

<header class="hero" id="ciel">
  <h1 class="hero__titre">KAZAFK</h1>
  <p class="hero__sous-titre">Un voyage à travers mes projets</p>
  <p class="hero__indice" aria-hidden="true">Descendre ▾</p>
</header>

<main id="etapes">
  <!-- Instantané statique : remplacé par les données fraîches de l'API GitHub quand elle répond -->
  <section class="etape" id="etape-1" style="--scene: url('assets/img/scene-profondeurs.jpg')">
    <div class="etape__fond" aria-hidden="true"></div>
    <article class="carte">
      <p class="carte__numero">Étape 01</p>
      <h2 class="carte__titre">sca-water-map</h2>
      <p class="carte__description">Un projet à découvrir.</p>
      <p class="carte__badges"><span class="badge">JavaScript</span></p>
      <a class="carte__lien" href="https://kazafk.github.io/sca-water-map/">Visiter le projet →</a>
    </article>
  </section>
  <section class="etape" id="etape-2" style="--scene: url('assets/img/scene-abysses.jpg')">
    <div class="etape__fond" aria-hidden="true"></div>
    <article class="carte">
      <p class="carte__numero">Étape 02</p>
      <h2 class="carte__titre">animated-octo-happiness</h2>
      <p class="carte__description">Un projet à découvrir.</p>
      <p class="carte__badges"><span class="badge">TypeScript</span></p>
      <a class="carte__lien" href="https://kazafk.github.io/animated-octo-happiness/">Visiter le projet →</a>
    </article>
  </section>
  <section class="etape" id="etape-3" style="--scene: url('assets/img/scene-profondeurs.jpg')">
    <div class="etape__fond" aria-hidden="true"></div>
    <article class="carte">
      <p class="carte__numero">Étape 03</p>
      <h2 class="carte__titre">mmia4</h2>
      <p class="carte__description">Informations MM</p>
      <p class="carte__badges"><span class="badge">HTML</span></p>
      <a class="carte__lien" href="https://kazafk.github.io/mmia4/">Visiter le projet →</a>
    </article>
  </section>
</main>

<footer class="fin" id="fin">
  <h2 class="fin__titre">Fin du voyage</h2>
  <p class="fin__texte">De nouveaux projets apparaîtront ici au fil de l'eau.</p>
  <p class="fin__actions">
    <a class="fin__lien" href="https://github.com/Kazafk">Profil GitHub ↗</a>
    <button class="fin__remonter" id="remonter" type="button">Remonter à la surface ↑</button>
  </p>
</footer>

<script src="https://cdn.jsdelivr.net/npm/gsap@3.12.5/dist/gsap.min.js" defer></script>
<script src="https://cdn.jsdelivr.net/npm/gsap@3.12.5/dist/ScrollTrigger.min.js" defer></script>
<script type="module" src="js/main.js"></script>
</body>
</html>
```

Note : `js/main.js` n'existe pas encore — un 404 console est attendu jusqu'à la Task 4. Le site statique fonctionne sans lui (c'est le mode « sans JavaScript » de la spec).

- [ ] **Step 2 : Écrire `css/styles.css`**

```css
:root {
  --or: #ecd06f;
  --turquoise: #49c5b6;
  --brun: #987654;
  --fond: #001714;
  --encre: #0c2724;
  --texte: #bfebe4;
  --police-titres: 'Playfair Display', serif;
  --police-textes: 'Hanken Grotesk', sans-serif;
}

* { margin: 0; padding: 0; box-sizing: border-box; }
html { scroll-behavior: smooth; }
body {
  background: var(--fond);
  color: var(--texte);
  font-family: var(--police-textes);
  line-height: 1.6;
  overflow-x: hidden;
}
body.curseur-actif,
body.curseur-actif a,
body.curseur-actif button { cursor: none; }

/* Voile de grain pictural (masque la douceur d'upscale des illustrations) */
body::after {
  content: '';
  position: fixed;
  inset: 0;
  z-index: 50;
  pointer-events: none;
  opacity: .08;
  mix-blend-mode: overlay;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='160' height='160'%3E%3Cfilter id='g'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='2'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23g)'/%3E%3C/svg%3E");
}

/* ---------- Hero : le ciel ---------- */
.hero {
  position: relative;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
  padding: 8vh 5vw;
  background:
    linear-gradient(rgba(0, 23, 20, .25), rgba(0, 23, 20, .6)),
    url('../assets/img/scene-ciel.jpg') center / cover no-repeat;
}
.hero__titre {
  font-family: var(--police-titres);
  font-weight: 900;
  font-size: clamp(3.5rem, 14vw, 9rem);
  letter-spacing: .08em;
  color: var(--or);
  text-shadow: 0 4px 40px rgba(0, 0, 0, .45);
}
.hero__sous-titre {
  font-size: clamp(1.05rem, 2.5vw, 1.5rem);
  font-style: italic;
  color: #f5ecd2;
  margin-top: 1rem;
}
.hero__indice {
  position: absolute;
  bottom: 4vh;
  color: var(--or);
  letter-spacing: .25em;
  text-transform: uppercase;
  font-size: .78rem;
  animation: flotter 2.2s ease-in-out infinite;
}
@keyframes flotter {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(8px); }
}

/* ---------- Étapes du voyage ---------- */
.etape {
  position: relative;
  min-height: 100vh;
  overflow: hidden;
  display: flex;
  align-items: center;
  padding: 12vh 8vw;
}
.etape__fond {
  position: absolute;
  inset: -14% 0; /* déborde verticalement pour la parallaxe */
  z-index: -2;
  background: var(--scene) center / cover no-repeat;
}
.etape::before {
  /* assombrissement pour la lisibilité des cartes */
  content: '';
  position: absolute;
  inset: 0;
  z-index: -1;
  background: linear-gradient(rgba(0, 23, 20, .55), rgba(0, 23, 20, .3) 45%, rgba(0, 23, 20, .65));
}
.etape:nth-child(odd) .carte { margin-right: auto; }
.etape:nth-child(even) .carte { margin-left: auto; }

.carte {
  max-width: 480px;
  width: 100%;
  background: rgba(0, 23, 20, .72);
  backdrop-filter: blur(6px);
  border: 1px solid rgba(236, 208, 111, .25);
  border-radius: 8px;
  padding: 2.2rem 2.4rem;
}
.carte__numero {
  color: var(--turquoise);
  font-size: .75rem;
  font-weight: 600;
  letter-spacing: .3em;
  text-transform: uppercase;
}
.carte__titre {
  font-family: var(--police-titres);
  font-weight: 700;
  color: var(--or);
  font-size: clamp(1.8rem, 4.5vw, 2.6rem);
  margin: .5rem 0;
  overflow-wrap: anywhere;
}
.carte__description { margin-bottom: 1.2rem; }
.carte__badges { margin-bottom: 1.6rem; }
.badge {
  display: inline-block;
  font-size: .75rem;
  font-weight: 600;
  letter-spacing: .05em;
  color: var(--or);
  border: 1px solid rgba(236, 208, 111, .45);
  border-radius: 999px;
  padding: .25rem .9rem;
}
.carte__lien {
  display: inline-block;
  background: var(--turquoise);
  color: var(--encre);
  font-weight: 600;
  text-decoration: none;
  border-radius: 999px;
  padding: .8rem 1.8rem;
  transition: transform .25s ease, box-shadow .25s ease;
}
.carte__lien:hover,
.carte__lien:focus-visible {
  transform: translateY(-3px);
  box-shadow: 0 12px 30px rgba(73, 197, 182, .35);
}

/* ---------- Fin du voyage ---------- */
.fin {
  min-height: 60vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
  padding: 14vh 5vw 10vh;
  background: linear-gradient(var(--fond), #000b09);
}
.fin__titre {
  font-family: var(--police-titres);
  color: var(--or);
  font-size: clamp(2rem, 6vw, 3rem);
}
.fin__texte {
  color: rgba(191, 235, 228, .7);
  margin: 1rem 0 2.2rem;
}
.fin__actions {
  display: flex;
  gap: 1rem;
  flex-wrap: wrap;
  justify-content: center;
}
.fin__lien,
.fin__remonter {
  font-family: var(--police-textes);
  font-size: 1rem;
  font-weight: 600;
  color: var(--texte);
  background: none;
  border: 1px solid rgba(73, 197, 182, .55);
  border-radius: 999px;
  padding: .8rem 1.8rem;
  text-decoration: none;
  transition: background .25s ease, color .25s ease;
}
.fin__lien:hover, .fin__lien:focus-visible,
.fin__remonter:hover, .fin__remonter:focus-visible {
  background: var(--turquoise);
  color: var(--encre);
}

/* ---------- Indicateur de progression ---------- */
.progression {
  position: fixed;
  right: 1.2rem;
  top: 50%;
  transform: translateY(-50%);
  z-index: 40;
  display: flex;
  flex-direction: column;
  gap: .7rem;
}
.progression a {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  border: 1px solid var(--or);
  display: block;
  opacity: .5;
  transition: all .3s ease;
}
.progression a.actif {
  background: var(--or);
  opacity: 1;
  transform: scale(1.25);
}

/* ---------- Curseur lumineux ---------- */
.curseur {
  position: fixed;
  left: 0;
  top: 0;
  width: 34px;
  height: 34px;
  margin: -17px;
  border-radius: 50%;
  z-index: 60;
  pointer-events: none;
  background: radial-gradient(circle, rgba(236, 208, 111, .85), rgba(236, 208, 111, 0) 65%);
  mix-blend-mode: screen;
  transition: width .2s ease, height .2s ease, margin .2s ease;
  display: none;
}
body.curseur-actif .curseur { display: block; }
.curseur--grand {
  width: 64px;
  height: 64px;
  margin: -32px;
}

/* ---------- Faune ---------- */
#faune {
  position: fixed;
  inset: 0;
  z-index: 4;
  pointer-events: none;
}

/* ---------- Mobile ---------- */
@media (max-width: 700px) {
  .progression { display: none; }
  .etape { padding: 10vh 6vw; }
  .etape:nth-child(odd) .carte,
  .etape:nth-child(even) .carte { margin: 0 auto; }
}

/* ---------- Mouvement réduit ---------- */
@media (prefers-reduced-motion: reduce) {
  html { scroll-behavior: auto; }
  .hero__indice { animation: none; }
  #faune, .curseur { display: none !important; }
}
```

- [ ] **Step 3 : Vérifier visuellement en local**

Run : `cd /c/Repos/Kazafk.github.io && python -m http.server 8000` (en arrière-plan), puis ouvrir `http://localhost:8000`.

Attendu :
- Hero plein écran : illustration ciel doré, « KAZAFK » massif en Playfair Display or, indice « Descendre ▾ » qui flotte en bas
- 3 étapes plein écran, fonds illustrés alternés (profondeurs / abysses / profondeurs), cartes alternées gauche/droite, lisibles
- Footer sombre avec les deux boutons arrondis
- Grain visible sur l'ensemble (texture fine)
- Aucun ascenseur horizontal ; console : seul le 404 de `js/main.js` est toléré à ce stade

- [ ] **Step 4 : Commit**

```bash
git add index.html css/styles.css
git commit -m "feat: fondation HTML/CSS du voyage (hero, étapes statiques, fin, grain)"
```

---

### Task 2 : Sélection des projets et échappement (`github.js`) — TDD

**Files:**
- Create: `tests/github.test.mjs`
- Create: `js/github.js`

- [ ] **Step 1 : Écrire les tests (qui échouent)**

Créer `tests/github.test.mjs` :

```js
import { test } from 'node:test';
import assert from 'node:assert/strict';
import { selectionnerProjets, echapper } from '../js/github.js';

const depot = (sur) => ({
  name: 'demo',
  has_pages: true,
  pushed_at: '2026-01-01T00:00:00Z',
  description: 'Desc',
  language: 'JS',
  ...sur,
});

test('garde uniquement les dépôts avec une Page, hors site lui-même', () => {
  const projets = selectionnerProjets([
    depot({ name: 'avec-page' }),
    depot({ name: 'sans-page', has_pages: false }),
    depot({ name: 'Kazafk.github.io' }),
  ]);
  assert.deepEqual(projets.map((p) => p.nom), ['avec-page']);
});

test('trie du plus récemment mis à jour au plus ancien', () => {
  const projets = selectionnerProjets([
    depot({ name: 'ancien', pushed_at: '2026-01-01T00:00:00Z' }),
    depot({ name: 'recent', pushed_at: '2026-06-01T00:00:00Z' }),
  ]);
  assert.deepEqual(projets.map((p) => p.nom), ['recent', 'ancien']);
});

test('construit l’URL de la GitHub Page du projet', () => {
  const [p] = selectionnerProjets([depot({ name: 'mon-projet' })]);
  assert.equal(p.url, 'https://kazafk.github.io/mon-projet/');
});

test('fournit une description par défaut quand elle manque', () => {
  const [p] = selectionnerProjets([depot({ description: null })]);
  assert.equal(p.description, 'Un projet à découvrir.');
});

test('expose le langage tel quel, ou null', () => {
  const [a] = selectionnerProjets([depot({ language: 'TypeScript' })]);
  const [b] = selectionnerProjets([depot({ language: null })]);
  assert.equal(a.langage, 'TypeScript');
  assert.equal(b.langage, null);
});

test('echapper neutralise les caractères HTML', () => {
  assert.equal(echapper(`<b>"&'`), '&lt;b&gt;&quot;&amp;&#39;');
});
```

- [ ] **Step 2 : Vérifier que les tests échouent**

Run : `node --test tests/`
Attendu : ÉCHEC — `Cannot find module ... js/github.js` (ERR_MODULE_NOT_FOUND).

- [ ] **Step 3 : Implémenter le minimum dans `js/github.js`**

```js
const UTILISATEUR = 'Kazafk';
const DEPOT_SITE = 'kazafk.github.io';

export function echapper(texte) {
  return String(texte).replace(/[&<>"']/g, (c) => ({
    '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;',
  }[c]));
}

export function selectionnerProjets(depots) {
  return depots
    .filter((d) => d.has_pages && d.name.toLowerCase() !== DEPOT_SITE)
    .sort((a, b) => new Date(b.pushed_at) - new Date(a.pushed_at))
    .map((d) => ({
      nom: d.name,
      description: d.description || 'Un projet à découvrir.',
      langage: d.language || null,
      url: `https://kazafk.github.io/${d.name}/`,
    }));
}

export { UTILISATEUR };
```

- [ ] **Step 4 : Vérifier que les tests passent**

Run : `node --test tests/`
Attendu : `pass 6` / `fail 0`.

- [ ] **Step 5 : Commit**

```bash
git add tests/github.test.mjs js/github.js
git commit -m "feat: sélection des projets GitHub Pages (filtre, tri, échappement)"
```

---

### Task 3 : Cache localStorage + appel API (`github.js`) — TDD

**Files:**
- Modify: `tests/github.test.mjs` (ajout en fin de fichier)
- Modify: `js/github.js` (ajout en fin de fichier)

- [ ] **Step 1 : Ajouter les tests (qui échouent)**

Ajouter à la fin de `tests/github.test.mjs` :

```js
import { lireCache, ecrireCache, recupererProjets } from '../js/github.js';

function fauxStockage() {
  const m = new Map();
  return {
    getItem: (k) => (m.has(k) ? m.get(k) : null),
    setItem: (k, v) => m.set(k, v),
  };
}

test('le cache restitue les projets avant une heure', () => {
  const stockage = fauxStockage();
  ecrireCache(stockage, 1000, [{ nom: 'p' }]);
  assert.deepEqual(lireCache(stockage, 1000 + 3599_000), [{ nom: 'p' }]);
});

test('le cache expire après une heure', () => {
  const stockage = fauxStockage();
  ecrireCache(stockage, 1000, [{ nom: 'p' }]);
  assert.equal(lireCache(stockage, 1000 + 3600_001), null);
});

test('le cache tolère un contenu corrompu', () => {
  const stockage = fauxStockage();
  stockage.setItem('kazafk-projets-v1', '{pas du json');
  assert.equal(lireCache(stockage, 0), null);
});

test('recupererProjets lit le cache sans appeler le réseau', async () => {
  const stockage = fauxStockage();
  ecrireCache(stockage, 1000, [{ nom: 'en-cache' }]);
  const fetchFn = () => { throw new Error('le réseau ne doit pas être appelé'); };
  const projets = await recupererProjets({ fetchFn, stockage, maintenant: 2000 });
  assert.deepEqual(projets.map((p) => p.nom), ['en-cache']);
});

test('recupererProjets interroge l’API puis écrit le cache', async () => {
  const stockage = fauxStockage();
  const fetchFn = async (url) => {
    assert.equal(url, 'https://api.github.com/users/Kazafk/repos?per_page=100');
    return { ok: true, json: async () => [depot({ name: 'frais' })] };
  };
  const projets = await recupererProjets({ fetchFn, stockage, maintenant: 1000 });
  assert.deepEqual(projets.map((p) => p.nom), ['frais']);
  assert.deepEqual(lireCache(stockage, 1000).map((p) => p.nom), ['frais']);
});

test('recupererProjets lève une erreur si la réponse est KO', async () => {
  const fetchFn = async () => ({ ok: false, status: 403 });
  await assert.rejects(
    recupererProjets({ fetchFn, stockage: fauxStockage(), maintenant: 0 }),
    /403/,
  );
});
```

- [ ] **Step 2 : Vérifier que les tests échouent**

Run : `node --test tests/`
Attendu : ÉCHEC — `lireCache` n'est pas exporté (SyntaxError d'import).

- [ ] **Step 3 : Implémenter dans `js/github.js`**

Ajouter à la fin du fichier :

```js
const CLE_CACHE = 'kazafk-projets-v1';
const TTL_CACHE_MS = 60 * 60 * 1000;

export function lireCache(stockage, maintenant) {
  try {
    const brut = stockage.getItem(CLE_CACHE);
    if (!brut) return null;
    const { horodatage, projets } = JSON.parse(brut);
    if (maintenant - horodatage > TTL_CACHE_MS) return null;
    return projets;
  } catch {
    return null;
  }
}

export function ecrireCache(stockage, maintenant, projets) {
  try {
    stockage.setItem(CLE_CACHE, JSON.stringify({ horodatage: maintenant, projets }));
  } catch {
    /* stockage indisponible (navigation privée) : on vivra sans cache */
  }
}

export async function recupererProjets({ fetchFn, stockage, maintenant } = {}) {
  fetchFn = fetchFn || fetch;
  stockage = stockage || localStorage;
  maintenant = maintenant ?? Date.now();

  const enCache = lireCache(stockage, maintenant);
  if (enCache) return enCache;

  const reponse = await fetchFn(`https://api.github.com/users/${UTILISATEUR}/repos?per_page=100`);
  if (!reponse.ok) throw new Error(`API GitHub : ${reponse.status}`);
  const projets = selectionnerProjets(await reponse.json());
  ecrireCache(stockage, maintenant, projets);
  return projets;
}
```

- [ ] **Step 4 : Vérifier que les tests passent**

Run : `node --test tests/`
Attendu : `pass 12` / `fail 0`.

- [ ] **Step 5 : Commit**

```bash
git add tests/github.test.mjs js/github.js
git commit -m "feat: cache localStorage (1 h) et récupération API avec injection de dépendances"
```

---

### Task 4 : Génération des étapes + orchestration (`htmlEtape`, `main.js`) — TDD

**Files:**
- Modify: `tests/github.test.mjs` (ajout en fin de fichier)
- Modify: `js/github.js` (ajout en fin de fichier)
- Create: `js/main.js`

- [ ] **Step 1 : Ajouter les tests (qui échouent)**

Ajouter à la fin de `tests/github.test.mjs` :

```js
import { htmlEtape } from '../js/github.js';

const projet = (sur) => ({
  nom: 'demo',
  description: 'Une description.',
  langage: 'JavaScript',
  url: 'https://kazafk.github.io/demo/',
  ...sur,
});

test('alterne les scènes profondeurs et abysses', () => {
  assert.match(htmlEtape(projet(), 0), /scene-profondeurs\.jpg/);
  assert.match(htmlEtape(projet(), 1), /scene-abysses\.jpg/);
  assert.match(htmlEtape(projet(), 2), /scene-profondeurs\.jpg/);
});

test('numérote et identifie chaque étape', () => {
  const html = htmlEtape(projet(), 1);
  assert.match(html, /Étape 02/);
  assert.match(html, /id="etape-2"/);
});

test('échappe les données venant de l’API', () => {
  const html = htmlEtape(projet({ nom: '<script>x</script>', description: 'a & b' }), 0);
  assert.match(html, /&lt;script&gt;/);
  assert.match(html, /a &amp; b/);
  assert.doesNotMatch(html, /<script>x/);
});

test('omet le badge quand le langage est inconnu', () => {
  assert.doesNotMatch(htmlEtape(projet({ langage: null }), 0), /class="badge"/);
});

test('relie l’étape à la GitHub Page du projet', () => {
  assert.match(htmlEtape(projet(), 0), /href="https:\/\/kazafk\.github\.io\/demo\/"/);
});
```

- [ ] **Step 2 : Vérifier que les tests échouent**

Run : `node --test tests/`
Attendu : ÉCHEC — `htmlEtape` n'est pas exporté.

- [ ] **Step 3 : Implémenter `htmlEtape` dans `js/github.js`**

Ajouter à la fin du fichier :

```js
export function htmlEtape(projet, indice) {
  const scene = indice % 2 === 0 ? 'scene-profondeurs.jpg' : 'scene-abysses.jpg';
  const numero = String(indice + 1).padStart(2, '0');
  const badge = projet.langage ? `<span class="badge">${echapper(projet.langage)}</span>` : '';
  return `
<section class="etape" id="etape-${indice + 1}" style="--scene: url('assets/img/${scene}')">
  <div class="etape__fond" aria-hidden="true"></div>
  <article class="carte">
    <p class="carte__numero">Étape ${numero}</p>
    <h2 class="carte__titre">${echapper(projet.nom)}</h2>
    <p class="carte__description">${echapper(projet.description)}</p>
    <p class="carte__badges">${badge}</p>
    <a class="carte__lien" href="${projet.url}">Visiter le projet →</a>
  </article>
</section>`;
}
```

- [ ] **Step 4 : Vérifier que les tests passent**

Run : `node --test tests/`
Attendu : `pass 17` / `fail 0`.

- [ ] **Step 5 : Écrire `js/main.js` (version données seules)**

```js
import { recupererProjets, htmlEtape } from './github.js';

async function demarrer() {
  const conteneur = document.getElementById('etapes');
  try {
    const projets = await recupererProjets();
    if (projets.length > 0) {
      conteneur.innerHTML = projets.map((p, i) => htmlEtape(p, i)).join('');
    }
  } catch (erreur) {
    // API en panne ou rate-limitée : l'instantané statique du HTML reste affiché
    console.warn('API GitHub indisponible, instantané statique conservé.', erreur);
  }
}

demarrer();
```

- [ ] **Step 6 : Vérifier en navigateur**

Avec le serveur local (`python -m http.server 8000`) :
1. Recharger `http://localhost:8000` — les 3 étapes s'affichent (désormais générées par l'API ; visuellement identiques à l'instantané).
2. DevTools → Network → cocher « Offline », vider `localStorage` (Application → Local Storage → supprimer `kazafk-projets-v1`), recharger : l'instantané statique reste affiché, le warning apparaît en console.
3. Décocher « Offline », recharger deux fois : au second chargement, aucun appel `api.github.com` dans l'onglet Network (cache 1 h actif).

- [ ] **Step 7 : Commit**

```bash
git add tests/github.test.mjs js/github.js js/main.js
git commit -m "feat: étapes générées depuis l'API GitHub avec fallback statique"
```

---

### Task 5 : Animations du voyage (`journey.js`)

**Files:**
- Create: `js/journey.js`
- Modify: `js/main.js`

- [ ] **Step 1 : Écrire `js/journey.js`**

```js
export function initVoyage() {
  const remonter = document.getElementById('remonter');
  if (remonter) {
    remonter.addEventListener('click', () => window.scrollTo({ top: 0, behavior: 'smooth' }));
  }

  construireProgression();

  const mouvementReduit = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
  if (mouvementReduit || typeof gsap === 'undefined') return; // contenu visible sans animation

  gsap.registerPlugin(ScrollTrigger);

  gsap.from('.hero__titre', { opacity: 0, y: 40, duration: 1.2, ease: 'power3.out' });
  gsap.from('.hero__sous-titre', { opacity: 0, y: 20, duration: 1, delay: .4, ease: 'power2.out' });

  document.querySelectorAll('.etape').forEach((etape) => {
    const fond = etape.querySelector('.etape__fond');
    if (fond) {
      gsap.fromTo(fond, { yPercent: -10 }, {
        yPercent: 10,
        ease: 'none',
        scrollTrigger: { trigger: etape, start: 'top bottom', end: 'bottom top', scrub: true },
      });
    }
    const carte = etape.querySelector('.carte');
    if (carte) {
      gsap.from(carte, {
        opacity: 0,
        y: 60,
        duration: .9,
        ease: 'power2.out',
        scrollTrigger: { trigger: etape, start: 'top 65%' },
      });
    }
  });
}

function construireProgression() {
  const nav = document.getElementById('progression');
  if (!nav) return;
  const sections = [
    { id: 'ciel', libelle: 'Le ciel' },
    ...[...document.querySelectorAll('.etape')].map((s, i) => ({ id: s.id, libelle: `Étape ${i + 1}` })),
    { id: 'fin', libelle: 'Fin du voyage' },
  ];
  nav.innerHTML = sections
    .map((s) => `<a href="#${s.id}" data-cible="${s.id}" aria-label="${s.libelle}"></a>`)
    .join('');

  const observateur = new IntersectionObserver((entrees) => {
    for (const entree of entrees) {
      if (entree.isIntersecting) {
        nav.querySelectorAll('a').forEach((a) => {
          a.classList.toggle('actif', a.dataset.cible === entree.target.id);
        });
      }
    }
  }, { rootMargin: '-45% 0px -45% 0px' });

  sections.forEach((s) => {
    const element = document.getElementById(s.id);
    if (element) observateur.observe(element);
  });
}
```

- [ ] **Step 2 : Brancher dans `js/main.js`**

Remplacer le contenu de `js/main.js` par :

```js
import { recupererProjets, htmlEtape } from './github.js';
import { initVoyage } from './journey.js';

async function demarrer() {
  const conteneur = document.getElementById('etapes');
  try {
    const projets = await recupererProjets();
    if (projets.length > 0) {
      conteneur.innerHTML = projets.map((p, i) => htmlEtape(p, i)).join('');
    }
  } catch (erreur) {
    // API en panne ou rate-limitée : l'instantané statique du HTML reste affiché
    console.warn('API GitHub indisponible, instantané statique conservé.', erreur);
  }
  initVoyage(); // après le rendu : les ScrollTriggers visent les étapes définitives
}

demarrer();
```

- [ ] **Step 3 : Vérifier en navigateur**

Sur `http://localhost:8000` :
1. Le titre et le sous-titre du hero apparaissent en fondu montant au chargement.
2. Au scroll : les fonds des étapes glissent (parallaxe), chaque carte surgit en fondu montant quand sa section entre.
3. Les points de progression à droite : 5 points (ciel + 3 étapes + fin), le point actif suit le scroll, un clic saute à la section.
4. « Remonter à la surface » ramène en haut en douceur.
5. DevTools → Rendering → « Emulate CSS prefers-reduced-motion: reduce » + rechargement : tout le contenu est immédiatement visible, aucune parallaxe, les points de progression fonctionnent toujours.
6. `node --test tests/` passe toujours (`pass 17`).

- [ ] **Step 4 : Commit**

```bash
git add js/journey.js js/main.js
git commit -m "feat: parallaxe et reveals GSAP, progression du voyage, retour surface"
```

---

### Task 6 : Faune animée (`fauna.js`)

**Files:**
- Create: `js/fauna.js`
- Modify: `js/main.js`

- [ ] **Step 1 : Écrire `js/fauna.js`**

Les créatures vivent en coordonnées document (oiseaux dans le premier écran, poissons en dessous) et sont projetées sur le canvas fixe selon le scroll.

```js
export function initFaune() {
  const canvas = document.getElementById('faune');
  if (!canvas || window.matchMedia('(prefers-reduced-motion: reduce)').matches) return;
  const ctx = canvas.getContext('2d');
  let creatures = [];

  function peupler() {
    const densite = window.innerWidth < 700 ? 6 : 14;
    const hauteurDoc = document.documentElement.scrollHeight;
    creatures = Array.from({ length: densite }, (_, i) => {
      const oiseau = i < Math.ceil(densite / 3);
      return {
        oiseau,
        x: Math.random() * window.innerWidth,
        y: oiseau
          ? 40 + Math.random() * window.innerHeight * 0.6
          : window.innerHeight + Math.random() * Math.max(1, hauteurDoc - window.innerHeight * 1.6),
        vitesse: (0.3 + Math.random() * 0.7) * (Math.random() < 0.5 ? -1 : 1),
        taille: oiseau ? 6 + Math.random() * 6 : 8 + Math.random() * 10,
        phase: Math.random() * Math.PI * 2,
      };
    });
  }

  function redimensionner() {
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
    peupler();
  }

  function dessinerOiseau(c, yEcran, t) {
    const battement = Math.sin(t / 200 + c.phase) * c.taille * 0.5;
    ctx.strokeStyle = 'rgba(12, 39, 36, .65)';
    ctx.lineWidth = 1.6;
    ctx.beginPath();
    ctx.moveTo(c.x - c.taille, yEcran + battement);
    ctx.quadraticCurveTo(c.x - c.taille / 2, yEcran - c.taille / 2, c.x, yEcran);
    ctx.quadraticCurveTo(c.x + c.taille / 2, yEcran - c.taille / 2, c.x + c.taille, yEcran + battement);
    ctx.stroke();
  }

  function dessinerPoisson(c, yEcran, t) {
    const ondulation = Math.sin(t / 600 + c.phase) * 6;
    const sens = c.vitesse > 0 ? 1 : -1;
    ctx.fillStyle = 'rgba(191, 235, 228, .4)';
    ctx.beginPath();
    ctx.ellipse(c.x, yEcran + ondulation, c.taille, c.taille * 0.45, 0, 0, Math.PI * 2);
    ctx.moveTo(c.x - sens * c.taille, yEcran + ondulation);
    ctx.lineTo(c.x - sens * (c.taille + 7), yEcran + ondulation - 5);
    ctx.lineTo(c.x - sens * (c.taille + 7), yEcran + ondulation + 5);
    ctx.closePath();
    ctx.fill();
  }

  function boucle(t) {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    for (const c of creatures) {
      c.x += c.vitesse;
      if (c.x < -40) c.x = canvas.width + 40;
      if (c.x > canvas.width + 40) c.x = -40;
      const yEcran = c.y - window.scrollY;
      if (yEcran < -40 || yEcran > canvas.height + 40) continue;
      if (c.oiseau) dessinerOiseau(c, yEcran, t);
      else dessinerPoisson(c, yEcran, t);
    }
    requestAnimationFrame(boucle);
  }

  redimensionner();
  window.addEventListener('resize', redimensionner);
  requestAnimationFrame(boucle);
}
```

- [ ] **Step 2 : Brancher dans `js/main.js`**

Ajouter l'import sous celui de `journey.js` :

```js
import { initFaune } from './fauna.js';
```

et appeler, juste après `initVoyage();` :

```js
  initFaune(); // après le rendu : la hauteur du document est définitive
```

- [ ] **Step 3 : Vérifier en navigateur**

1. Hero : silhouettes d'oiseaux sombres qui battent des ailes et traversent l'écran.
2. Étapes : poissons translucides ondulant à différentes profondeurs (ils défilent avec le scroll).
3. DevTools mode mobile (375 px) : moins de créatures, pas de saccade au scroll.
4. Emulation reduced-motion + rechargement : aucune créature.

- [ ] **Step 4 : Commit**

```bash
git add js/fauna.js js/main.js
git commit -m "feat: faune ambiante en canvas (oiseaux du ciel, poissons des profondeurs)"
```

---

### Task 7 : Curseur lumineux (`cursor.js`)

**Files:**
- Create: `js/cursor.js`
- Modify: `js/main.js`

- [ ] **Step 1 : Écrire `js/cursor.js`**

```js
export function initCurseur() {
  const curseur = document.getElementById('curseur');
  if (!curseur) return;
  const tactile = window.matchMedia('(pointer: coarse)').matches;
  const mouvementReduit = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
  if (tactile || mouvementReduit) {
    curseur.remove();
    return;
  }

  document.body.classList.add('curseur-actif');
  let cibleX = window.innerWidth / 2;
  let cibleY = window.innerHeight / 2;
  let x = cibleX;
  let y = cibleY;

  window.addEventListener('mousemove', (e) => {
    cibleX = e.clientX;
    cibleY = e.clientY;
  });

  // Grossit sur les éléments interactifs (délégation : survit aux re-rendus)
  document.addEventListener('mouseover', (e) => {
    curseur.classList.toggle('curseur--grand', Boolean(e.target.closest('a, button')));
  });

  (function boucle() {
    x += (cibleX - x) * 0.15;
    y += (cibleY - y) * 0.15;
    curseur.style.transform = `translate(${x}px, ${y}px)`;
    requestAnimationFrame(boucle);
  })();
}
```

- [ ] **Step 2 : Brancher dans `js/main.js`**

Ajouter l'import sous celui de `fauna.js` :

```js
import { initCurseur } from './cursor.js';
```

et appeler, juste après `initFaune();` :

```js
  initCurseur();
```

- [ ] **Step 3 : Vérifier en navigateur**

1. Une lueur dorée suit la souris avec une légère inertie ; le curseur natif est masqué.
2. Elle grossit au survol des liens « Visiter le projet », des boutons du footer et des points de progression.
3. DevTools mode tactile (toolbar device) + rechargement : pas de lueur, curseur natif normal.
4. Emulation reduced-motion + rechargement : idem.

- [ ] **Step 4 : Commit**

```bash
git add js/cursor.js js/main.js
git commit -m "feat: curseur lumineux avec inertie, désactivé tactile et reduced-motion"
```

---

### Task 8 : Passe accessibilité et performance

**Files:**
- Modify: `index.html`, `css/styles.css` (selon constats)

- [ ] **Step 1 : Vérifications manuelles d'accessibilité**

Sur `http://localhost:8000` :
1. Parcours **clavier seul** : Tab atteint chaque « Visiter le projet », les points de progression, « Profil GitHub », « Remonter à la surface » ; le focus est visible sur chacun (sinon ajouter dans `styles.css` : `a:focus-visible, button:focus-visible { outline: 2px solid var(--or); outline-offset: 3px; }`).
2. Zoom navigateur 200 % : aucun texte coupé, pas d'ascenseur horizontal.
3. Vérifier le contraste des textes des cartes sur l'outil DevTools (inspecteur de couleur) : ratio ≥ 4.5:1 — la carte a un fond sombre à 72 % d'opacité qui le garantit ; sinon monter l'opacité à .85.

- [ ] **Step 2 : Lighthouse**

DevTools → Lighthouse → catégories Performance + Accessibilité + Best Practices, mode Navigation, appareil Desktop puis Mobile.
Attendu : Performance ≥ 90, Accessibilité ≥ 90. Corriger ce que Lighthouse signale (cas probables et remèdes) :
- « Image elements do not have explicit width and height » → non applicable (fonds CSS) ;
- « Largest Contentful Paint » lent → ajouter `<link rel="preload" as="image" href="assets/img/scene-ciel.jpg">` dans le `<head>` ;
- fonts qui bloquent → déjà en `display=swap`.

- [ ] **Step 3 : Re-tester et commit**

Run : `node --test tests/` → `pass 17`.

```bash
git add -A
git commit -m "fix: focus visible, preload du hero et correctifs Lighthouse"
```

(Si aucun correctif n'a été nécessaire, sauter le commit.)

---

### Task 9 : Mise en ligne sur GitHub Pages

**Files:** aucun nouveau fichier (opérations git/API).

- [ ] **Step 1 : Créer le dépôt distant `Kazafk/Kazafk.github.io`**

Le token GitHub est dans le Credential Manager Windows (compte `Kazafk`).

```bash
TOKEN=$(printf "protocol=https\nhost=github.com\n\n" | git credential fill | grep '^password=' | cut -d= -f2-)
curl -s -X POST https://api.github.com/user/repos \
  -H "Authorization: Bearer $TOKEN" \
  -H "Accept: application/vnd.github+json" \
  -d '{"name":"Kazafk.github.io","description":"Site vitrine — Le Voyage à travers mes projets","homepage":"https://kazafk.github.io"}' | head -c 400
```

Attendu : JSON avec `"full_name": "Kazafk/Kazafk.github.io"`. Si `422` avec « name already exists », le dépôt existe déjà : passer à l'étape suivante.

- [ ] **Step 2 : Pousser `main`**

```bash
cd /c/Repos/Kazafk.github.io
git remote add origin https://github.com/Kazafk/Kazafk.github.io.git
git push -u origin main
```

Attendu : `main -> main` sans erreur.

- [ ] **Step 3 : Activer GitHub Pages (branche main, racine)**

Pour un dépôt `<user>.github.io`, Pages s'active souvent seul ; cette commande force la configuration :

```bash
curl -s -X POST https://api.github.com/repos/Kazafk/Kazafk.github.io/pages \
  -H "Authorization: Bearer $TOKEN" \
  -H "Accept: application/vnd.github+json" \
  -d '{"source":{"branch":"main","path":"/"}}' | head -c 300
```

Attendu : `201` (créé) ou `409` (« already exists » — déjà actif, c'est bon).

- [ ] **Step 4 : Vérifier le site en ligne**

```bash
for i in $(seq 1 20); do
  CODE=$(curl -s -o /dev/null -w "%{http_code}" https://kazafk.github.io)
  echo "essai $i : HTTP $CODE"
  [ "$CODE" = "200" ] && break
  sleep 15
done
```

Attendu : `HTTP 200` (le premier déploiement peut prendre 2-3 minutes).
Puis ouvrir https://kazafk.github.io dans le navigateur : voyage complet fonctionnel, les 3 « Visiter le projet » débranchent vers les GitHub Pages respectives.

- [ ] **Step 5 : Vérification finale et clôture**

1. `node --test tests/` → `pass 17`.
2. Sur le site en ligne : Network → l'appel `api.github.com/users/Kazafk/repos` répond `200` et les étapes affichées correspondent aux dépôts réels.
3. Test mobile réel ou simulé : scroll fluide, pas de curseur custom.

---

## Auto-revue effectuée

- **Couverture spec :** direction artistique (Task 1), structure hero/étapes/fin (Task 1, 4), API + cache + tri (Task 2-3), fallback statique (Task 1, 4), GSAP parallaxe/reveals + progression + remonter (Task 5), faune (Task 6), curseur (Task 7), reduced-motion/mobile/sans-JS (Tasks 1, 5-7), grain (Task 1), footer sans mail/Twitter (Task 1), Lighthouse (Task 8), déploiement + activation Pages (Task 9). Aucun écart.
- **Placeholders :** aucun — chaque étape contient le code ou la commande exacte.
- **Cohérence des types :** `projet = { nom, description, langage, url }` constant entre `selectionnerProjets` (Task 2), les tests, `htmlEtape` (Task 4) et `main.js` ; signatures `initVoyage()`, `initFaune()`, `initCurseur()` sans argument, cohérentes entre définition et appels.
