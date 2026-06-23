# Kazafk — Portfolio

Portfolio personnel mono-page, hébergé sur **GitHub Pages** :
👉 **https://kazafk.github.io**

Thème **« maps, sand & stones »** : esthétique cartographique, palette sable /
terre cuite / pierre, fond de **courbes de niveau animées** (marching squares en
canvas) et en-tête qui fait défiler les sommets de France les plus proéminents.
Les projets sont chargés dynamiquement depuis l'**API GitHub**.

## Stack

- HTML / CSS / JS **vanilla**, tout en un seul fichier (`index.html`)
- [GSAP](https://gsap.com/) + ScrollTrigger (animations, via CDN)
- Polices : [Fraunces](https://fonts.google.com/specimen/Fraunces) + [JetBrains Mono](https://fonts.google.com/specimen/JetBrains+Mono)
- Aucune étape de build

## Développement local

```bash
python3 -m http.server 8000
# http://localhost:8000
```

## Déploiement

Push sur `master` → GitHub Pages publie automatiquement.

## Structure

```
index.html   # site complet (HTML + CSS + JS inline)
README.md    # ce fichier
CLAUDE.md    # contexte projet pour Claude Code
```

## Crédits données

Liste des sommets : [Wikipédia — Liste des sommets de France les plus proéminents](https://fr.wikipedia.org/wiki/Liste_des_sommets_de_France_les_plus_pro%C3%A9minents).
