# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Présentation du projet

Site statique Hugo — blog de recherche personnel de Mattéo Vacher documentant ses simulations de colonies de fourmis et l'IA. Hébergé sur `https://matteovacher.github.io/` et déployé via GitHub Actions sur GitHub Pages.

## Commandes

```bash
# Servir localement avec rechargement automatique (brouillons exclus)
hugo server

# Servir localement en incluant les brouillons
hugo server -D

# Construire pour la production
hugo build --gc --minify

# Créer un nouveau post
hugo new posts/mon-titre.md

# Créer une nouvelle page de ressources
hugo new ressources/ma-ressource.md
```

Hugo v0.156.0 édition étendue est requise (version utilisée exactement par la CI).

## Architecture

- **Thème** : PaperMod via sous-module git dans `themes/PaperMod/`. Les surcharges de thème vont dans `layouts/partials/`.
- **Rendu mathématique** : KaTeX chargé via CDN (`layouts/partials/extend_head.html` + `math.html`). À activer par post avec `math = true` dans le front matter.
- **CSS personnalisé** : `assets/css/extended/custom.css` — pris en charge automatiquement par le pipeline d'assets Hugo.
- **Recherche** : Fuse.js côté client. La page `search.md` doit avoir `layout: search` et le format de sortie JSON doit rester configuré dans `hugo.toml`.
- **CI/CD** : `.github/workflows/hugo.yaml` construit et déploie à chaque push sur `main`. Les sous-modules sont récupérés récursivement.

## Front matter des contenus

Front matter TOML (utiliser les délimiteurs `+++`, conformément à l'archétype) :

```toml
+++
date = '2026-03-28T00:00:00+01:00'
draft = false
title = 'Titre du post'
math = true        # uniquement si le post contient du LaTeX
summary = 'Résumé affiché dans les listes de posts.'
+++
```

Les posts avec `draft = true` ne sont pas construits en production — supprimer la ligne ou passer à `false` avant de publier.

## Sections de contenu

- `content/posts/` — Posts de recherche techniques (incluent souvent des maths KaTeX)
- `content/ressources/` — Résumés de cours et références
- `content/about.md` — Page de biographie personnelle
- `content/search.md` — Page de recherche (ne pas modifier le front matter)
