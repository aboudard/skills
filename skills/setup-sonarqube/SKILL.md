---
name: setup-sonarqube
description: Configure l’analyse d’un dépôt avec SonarQube Cloud ou un serveur SonarQube existant. À utiliser pour intégrer SonarQube à la CI, choisir un scanner, préparer un projet ou configurer les paramètres GitHub associés.
---

# Configurer SonarQube

Prépare l’analyse d’un dépôt avec SonarQube Cloud ou un serveur SonarQube existant. Le résultat attendu est une intégration minimale et vérifiable, accompagnée des étapes que seul l’utilisateur peut effectuer.

## 1. Examiner le dépôt

Depuis la racine du dépôt, repère :

- les langages, frameworks, outils de build, tests et rapports de couverture réellement présents ;
- les workflows CI existants, leurs déclencheurs, les commandes de build et les conventions de versions d’actions ;
- les configurations Sonar existantes (`sonar-project.properties`, propriétés du build, paramètres dans la CI) ;
- les modules et chemins pertinents si le dépôt est un monorepo.

Réutilise le build et la CI existants. Identifie les configurations déjà fonctionnelles avant de proposer des changements ; ne remplace pas un workflow complet lorsqu’une modification ciblée suffit.

**Terminé quand** les modules à analyser, le scanner déjà en place, les rapports disponibles et les points d’intégration possibles sont établis, ou que toute inconnue empêchant un choix est explicitement signalée.

## 2. Choisir la plateforme et confirmer les prérequis

Détermine si le dépôt doit envoyer ses analyses vers **SonarQube Cloud** ou vers un **serveur SonarQube existant**. Utilise la configuration trouvée comme indice ; si la destination reste incertaine, présente brièvement les deux options et demande à l’utilisateur de choisir avant de configurer la connexion.

- Un projet doit exister sur la plateforme pour recevoir les analyses.
- SonarQube Server doit être accessible depuis le runner CI ; signale les contraintes réseau ou de certificat visibles dans le dépôt.
- La création d’un compte, d’un projet ou d’un serveur peut nécessiter une action humaine. Guide l’utilisateur vers la plateforme choisie ; ne provisionne ni ne déploie un serveur implicitement.

**Terminé quand** la destination et les prérequis d’accès sont connus, ou que les actions humaines nécessaires sont listées sans prétendre qu’elles sont déjà accomplies.

## 3. Choisir le scanner adapté

Privilégie le scanner déjà utilisé par le projet. Sinon, consulte la documentation SonarSource actuelle pour choisir un scanner compatible avec le langage et le build :

- Maven, Gradle et .NET ont leurs scanners dédiés ;
- pour les autres projets compatibles, l’action officielle `SonarSource/sonarqube-scan-action` est une option ;
- certains langages ou builds natifs exigent une étape de compilation, un wrapper ou une configuration particulière. Vérifie leurs prérequis avant de choisir.

Vérifie les versions, entrées et paramètres dans la documentation officielle au moment du changement. Respecte la politique de pinning des actions déjà suivie par le dépôt ; n’introduis pas de version supposée à jour depuis un exemple ancien.

**Terminé quand** le scanner est adapté au build réel, ses prérequis sont satisfaits ou clairement signalés, et son emplacement dans la CI existante est défini.

## 4. Préparer une adoption progressive

Fais porter l’adoption sur le nouveau code lorsque la plateforme et le projet le permettent. Préserve les paramètres et règles existants ; ne crée pas de nouveaux seuils de qualité ni de conditions de fusion par défaut.

- Récupère l’historique Git complet lorsque le scanner en a besoin pour l’analyse des branches et des pull requests.
- Utilise uniquement les rapports de tests et de couverture que le projet génère réellement. Vérifie leur format et leur chemin ; ne fabrique pas de rapport, de valeur ou de seuil pour rendre une analyse « verte ».
- N’ajoute des exclusions que pour des chemins identifiés comme générés, vendoriés ou hors périmètre, et indique la raison de chaque exclusion. Ne masque pas le code ou les tests pour contourner une métrique.
- Une analyse réussie et un quality gate réussi sont deux résultats distincts. Un quality gate existant peut inclure des conditions de couverture même si aucun rapport n’est encore produit ; signale ce cas sans inventer de couverture.
- Demande confirmation avant de rendre un quality gate bloquant dans la CI ou d’indiquer qu’un contrôle Sonar est obligatoire avant fusion. Les règles de protection de branche se configurent séparément dans GitHub.

**Terminé quand** le périmètre d’analyse reflète le dépôt, les rapports configurés sont réels et les conditions de blocage éventuelles sont explicitement confirmées par l’utilisateur.

## 5. Configurer la CI et les paramètres

Modifie le workflow existant lorsque c’est possible ; crée un workflow dédié seulement si le dépôt n’en a pas d’adapté. Fais suivre les déclencheurs et les commandes de build du projet plutôt que d’imposer une nouvelle convention.

Configure les paramètres de connexion selon la destination :

- **Cloud** : le token `SONAR_TOKEN` est un secret GitHub. Les clés d’organisation et de projet dépendent du projet Sonar et doivent provenir de sa configuration.
- **Serveur** : le token `SONAR_TOKEN` est un secret GitHub ; `SONAR_HOST_URL` est l’URL du serveur et doit être configurée comme variable ou paramètre selon les conventions du dépôt.

N’affiche, ne lis et ne demande jamais à l’utilisateur de coller la valeur du token dans le dépôt, le workflow, un journal ou la conversation. L’utilisateur crée le token dans SonarQube puis le saisit lui-même dans **Settings > Secrets and variables > Actions** (ou au niveau d’organisation/environnement approprié). Ne crée pas de secret à sa place et ne fais pas échouer la configuration locale au seul motif qu’il n’est pas encore défini.

Garde les événements de pull request compatibles avec la politique de sécurité du dépôt. Les secrets ne sont pas transmis aux workflows de pull request provenant de forks ; explique les limites de l’analyse dans ce cas. N’utilise pas `pull_request_target` pour contourner l’absence de secrets.

Place chaque paramètre à un seul endroit faisant autorité et évite les propriétés Sonar redondantes ou contradictoires. Préfère la configuration centralisée de la plateforme quand elle convient ; conserve dans le dépôt les valeurs nécessaires à l’identification du projet et au fonctionnement reproductible du scanner.

**Terminé quand** le diff ne contient que les changements nécessaires, les valeurs non secrètes et les noms de variables sont cohérents, et les actions manuelles sont séparées clairement des changements réalisés.

## 6. Vérifier et rendre compte

Valide la syntaxe du workflow avec les outils disponibles et vérifie que les chemins de build, de sources, de tests et de rapports correspondent au dépôt. Lance les validations locales pertinentes sans prétendre qu’elles exécutent une analyse distante.

Termine en indiquant :

- les fichiers modifiés et la plateforme/scanner retenus ;
- les vérifications effectuées et les éventuels blocages ;
- les actions humaines restantes (projet, token, URL, accès réseau ou paramètres de plateforme) ;
- si une analyse distante a réellement réussi. Ne l’affirme que si son exécution CI et son résultat sont observables.

**Terminé quand** les changements sont validés au niveau local possible et que l’utilisateur peut distinguer clairement la configuration prête d’une analyse distante effectivement exécutée.

## Références officielles

Consulte ces sources au moment de configurer le dépôt ; les exemples et versions peuvent évoluer :

- [SonarQube Cloud avec GitHub Actions](https://docs.sonarsource.com/sonarqube-cloud/analyzing-source-code/ci-based-analysis/github-actions-for-sonarcloud/)
- [SonarQube Server avec GitHub Actions](https://docs.sonarsource.com/sonarqube-server/analyzing-source-code/ci-integration/github-actions/)
- [Action officielle SonarQube Scan](https://github.com/SonarSource/sonarqube-scan-action)
- [Configuration des paramètres d’analyse Cloud](https://docs.sonarsource.com/sonarqube-cloud/analyzing-source-code/analysis-parameters/configuration-overview/)
- [Quality gates Cloud](https://docs.sonarsource.com/sonarqube-cloud/standards/managing-quality-gates/introduction-to-quality-gates/)
- [Secrets GitHub Actions](https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-secrets)
