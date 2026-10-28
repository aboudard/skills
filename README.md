# GitHub Pages Setup Skill

Ce dépôt contient des **skills** normalisés (selon la norme ouverte [agentskills.io](https://agentskills.io)) pour automatiser des tâches de développement à l'aide de votre assistant IA.

Le premier skill disponible est **`setup-github-pages`**.

---

## 🚀 Ce que fait le skill `setup-github-pages`

Lorsqu'il est invoqué sur un projet, ce skill analyse automatiquement sa structure à la racine pour détecter sa pile technique :
1. **Angular** : Détecte `angular.json` ou `@angular/core`.
2. **Vite (React / Vue / SPA)** : Détecte `vite.config.ts` ou `vite.config.js`.
3. **HTML / CSS / JS Statique pur** : Propose un déploiement direct de fichiers statiques.

Il génère ensuite automatiquement le workflow GitHub Actions correspondant dans `.github/workflows/deploy.yml` et vous guide sur la configuration de la source de déploiement.

---

## 📥 Installation

Vous pouvez installer ce skill de deux manières différentes selon vos besoins (globalement sur votre machine ou localement dans un projet spécifique) à l'aide de l'outil `npx skills`.

### 1. Installation Globale (Recommandée)
Idéal si vous gérez de nombreux projets locaux (par exemple dans `C:/github`) et que vous souhaitez que l'assistant IA dispose de ce skill sur toute votre machine sans avoir à le réinstaller partout.

```bash
npx skills@latest add aboudard/skills --skill=setup-github-pages --global
```
*(Remplacez `aboudard/skills` par le nom de votre dépôt GitHub public, ex: `mon-pseudo/mes-skills`).*

### 2. Installation Locale (Par projet)
Idéal si vous souhaitez que le skill soit attaché uniquement à un dépôt spécifique (et versionné avec lui pour vos collaborateurs). Placez-vous à la racine de votre projet local puis exécutez :

```bash
npx skills@latest add aboudard/skills --skill=setup-github-pages
```

---

## 🛠️ Structure du dépôt

```text
.
├── README.md
└── skills
    └── setup-github-pages
        └── SKILL.md
```