---
name: setup-github-pages
description: Configure automatiquement un workflow GitHub Actions pour déployer le projet local sur GitHub Pages en détectant sa stack technique (Vite, Angular ou HTML statique).
---

# Skill : Setup GitHub Pages

Ce skill automatise l'ajout ou la mise à jour d'un workflow de déploiement GitHub Pages (`.github/workflows/deploy.yml`) adapté à la pile technique du dépôt courant.

## Étape 1 : Détection de la stack technique
Analyse la structure du projet à la racine (`package.json`, fichiers de configuration) pour identifier son type :
1. **Angular** : Présence de `angular.json` ou `@angular/core` dans le `package.json`.
2. **Vite (React / Vue / SPA)** : Présence de `vite.config.ts` ou `vite.config.js`.
3. **HTML/CSS/JS Statique pur** : Absence de bundler ou de framework (fichiers HTML directs à la racine ou dans un dossier `public`).

---

## Étape 2 : Application du template correspondant

Une fois la stack identifiée, crée le dossier `.github/workflows/` si nécessaire et écris le fichier `deploy.yml` en choisissant le bloc ci-dessous :

### Option 1 : Projet Angular
> **Note :** Pour Angular, il faut généralement configurer le build en production et ajuster le chemin de sortie (souvent `dist/nom-du-projet/browser` ou `dist/`). Vérifie le nom du projet dans le `package.json` ou `angular.json`.

```yaml
name: Deploy Angular to GitHub Pages

on:
  push:
    branches: [ main, master ]

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Node
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build Angular
        run: npm run build -- --configuration production

      - name: Setup Pages
        uses: actions/configure-pages@v4

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: './dist/' 

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### Option 2 : Projet Vite (React / Vue / SPA)

```yaml
name: Deploy Vite to GitHub Pages

on:
  push:
    branches: [ main, master ]

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Node
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build

      - name: Setup Pages
        uses: actions/configure-pages@v4

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: './dist' 

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### Option 3 : Site HTML / CSS / JS Statique pur

```yaml
name: Deploy Static HTML to GitHub Pages

on:
  push:
    branches: [ main, master ]

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: true

jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Pages
        uses: actions/configure-pages@v4

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: '.' 

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

---

## Étape 3 : Validation finale
1. Confirme à l'utilisateur quel template a été appliqué en fonction de la détection.
2. Rappelle-lui de vérifier dans les paramètres GitHub de son dépôt (**Settings > Pages**) que la source de déploiement est bien positionnée sur **GitHub Actions**.
3. Informe l'utilisateur que le site devrait être accessible à l'URL indiquée dans la section **GitHub Pages** des paramètres du dépôt.