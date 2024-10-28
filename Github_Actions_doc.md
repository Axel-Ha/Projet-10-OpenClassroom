# Documentation des GitHub Actions pour le projet BobApp

## 1. Introduction
Le repository GitHub contient deux workflows, l'un pour le CI et le second pour le CD. Ces workflows permettent d'automatiser la compilation, les tests, la génération du rapport de couverture de code par les tests, l'analyse de la qualité de code, ainsi que la génération et le déploiement des images Docker.  
Ce document a pour objectif de détailler chaque étape des GitHub Actions, de proposer des KPIs et de rapporter les premières métriques collectées après l'exécution des pipelines. Le document inclura également l'analyse des retours d'utilisateurs pertinents, de manière à identifier les améliorations nécessaires à mettre en place en priorité.  

## 2. Liens vers le projet SonarCloud et le repository DockerHub

### [SonarCloud](https://sonarcloud.io/project/overview?id=axel-ha_bobapp)  

### [DockerHub](https://hub.docker.com/repositories/kidax)    

## 3. Etapes des GitHub Actions

## Pipeline CI
Fichier: `CI.yml`  
Emplacement: `.github/workflows`

### Déclenchement du workflow 
``` yaml
on:
  pull_request:
  push:
    paths:
      - 'front/**'
      - 'back/**'
      - '.github/workflows/**'
    branches: [main]
```
- **Push** sur les chemins `front/**`, `back/**` et `.github/workflows/**` dans la branche main.
- **Pull request** sur les chemins `front/**`, `back/**` et `.github/workflows/**` dans la branche main.

**Objectifs**: Préciser comment le workflow va se déclencher en spécifiant le chemin et la branche du projet.

### Jobs
1. **`build_test_and_analyze`**

**Objectifs**: Builder le backend ainsi que le frontend, exécuter les tests, générer le rapport de couverture et analyser la qualité du code avec SonarCloud.

### Etapes:

1.1. **Set up environment**
``` yaml
  runs-on: ubuntu-latest
```
**Objectifs**: Indique le système d'exploitation de l'environnement virtuel dans lequel le job sera exécuté.

1.2. **Checkout code**
``` yaml
- name: Checkout code
  uses: actions/checkout@v4
```
**Objectif**: Récupération du code source du repository.
___

1.3. **Set up JDK 17**
``` yaml
- name: Set up JDK 17
  uses: actions/setup-java@v4
  with:
    java-version: '17'
    distribution: 'temurin'
```
**Objectif**: Installer JDK 17, qui sera nécessaire pour la compilation du code et l'exécution des tests.
___

1.4. **Cache Maven packages**
``` yaml
- name: Cache Maven packages
  uses: actions/cache@v4
  with:
    path: ~/.m2
    key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
    restore-keys: ${{ runner.os }}-m2
```
**Objectif**: Mise en cache des dépendances Maven pour accélérer le build.
___

1.5. **Set up Node.js**
``` yaml
- name: Set up Node.js
  uses: actions/setup-node@v4
  with:
    node-version: '18'
```
**Objectif**: Installation de Node.js, qui sera nécessaire pour le build du frontend.
___

1.6. **Cache Node.js modules**
``` yaml
- name: Cache Node.js modules
  uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-
```
**Objectif**: Mise en cache des modules Node.js pour accélérer les installations.
___

1.7. **Install frontend dependencies and run tests**
``` yaml
- name: Install frontend dependencies and run tests
  run: npm install && npm run test -- --code-coverage --browsers=ChromeHeadless --watch=false
```
**Objectif**: Installation des dépendances, éxécution des tests et génération d'un rapport de couverture.
___

1.8. **Build and verify backend with Maven**
``` yaml
- name: Build and verify backend with Maven
  run: mvn verify
```
**Objectif**: Compilation du backend et éxécution des tests et génération d'un rapport de couverture.
___

1.9. **Commit and push coverage reports**
``` yaml
- name: Commit and push coverage reports
  env:
    PAT: ${{ secrets.PAT }}
  run: |
       git config --global user.name 'github-actions'
       git config --global user.email 'github-actions@github.com'
       git remote set-url origin https://github.com/Axel-Ha/Projet-7-OpenClassroom.git
       git add -f ./front/coverage/bobapp/
       git add -f ./back/target/site/jacoco/jacoco.xml
       git commit -m "Add coverage reports [skip ci]"
       git push https://x-access-token:${{ secrets.PAT }}@github.com/Axel-Ha/Projet-7-OpenClassroom.git HEAD:main
```
**Objectif**: Sauvegarder et envoyer les rapports de couverture de code générés durant les tests au dépôt Git.
___

1.10 **Cache SonarQube packages**
``` yaml
- name: Cache SonarQube packages
  uses: actions/cache@v4
  with:
    path: ~/.sonar/cache
    key: ${{ runner.os }}-sonar
    restore-keys: ${{ runner.os }}-sonar
```
**Objectif**: Mise en cache des paquets SonarQube pour optimiser le temps d'exécution des analyses.
___

1.11 **Analyze with SonarCloud**
``` yaml
- name: Analyze with SonarCloud
  with:
    projectBaseDir: .
    args: >
        -Dsonar.projectKey=axel-ha_bobapp
        -Dsonar.organization=axel-ha
        -X
```
**Objectif**: Analyse du code avec SonarCloud pour vérifier sa qualité et détecter d'éventuels bugs ou vulnérabilités.


## Pipeline CD
Fichier: `CD.yml`  
Emplacement: `.github/workflows`

### Déclenchement du workflow 
``` yaml
on:
  pull_request:
  push:
    paths:
      - 'front/**'
      - 'back/**'
      - '.github/workflows/**'
    branches: [main]
```
- **Push** sur les chemins `front/**`, `back/**` et `.github/workflows/**` dans la branche main.
- **Pull request** sur les chemins `front/**`, `back/**` et `.github/workflows/**` dans la branche main.

**Objectifs**: Préciser comment le workflow va se déclencher en spécifiant le chemin et la branche du projet.

### Jobs
1. **`build_and_push_docker_image`**

**Objectifs**: Builder et déployer une image Docker du backend et frontend sur DockerHub.

### Etapes:

1.1. **Set up environment**
``` yaml
  runs-on: ubuntu-latest
```
**Objectifs**: Indique le système d'exploitation de l'environnement virtuel dans lequel le job sera exécuté.

1.2. **Checkout code**
``` yaml
- name: Checkout code
  uses: actions/checkout@v4
```
**Objectif**: Récupération du code source pour la construction de l'image Docker.
___

1.3. **Set up JDK 17**
``` yaml
- name: Set up JDK 17
  uses: actions/setup-java@v4
  with:
    java-version: '17'
    distribution: 'temurin'
```
**Objectif**: Installer JDK 17, qui sera nécessaire pour la compilation du code et l'exécution des tests.
___

1.4. **Login to Docker Hub**
``` yaml
- name: Login to Docker Hub
  uses: docker/login-action@v3
  with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_PASSWORD }}

```
**Objectif**: Authentification à Docker Hub pour permettre le déploiement de l'image Docker.
___

1.5. **Build Docker images**
``` yaml
- name: Build Docker images
  run: |
        docker build -t ${{ secrets.DOCKERHUB_USERNAME }}/bobapp-backend ./back
        docker build -t ${{ secrets.DOCKERHUB_USERNAME }}/bobapp-frontend ./front
          
```
**Objectif**: Construire les images Docker pour le backend et le frontend.
___

1.6. **Push Docker images**
``` yaml
- name: Push Docker images
  run: |
        docker push ${{ secrets.DOCKERHUB_USERNAME }}/bobapp-backend:latest
        docker push ${{ secrets.DOCKERHUB_USERNAME }}/bobapp-frontend:latest
          
```
**Objectif**: Pousser les images Docker du backend et du frontend sur Docker Hub pour les rendre disponibles pour le déploiement.

## 4. Proposition de KPIs
Pour garantir l'amélioration continue de la qualité de BobApp, je propose 4 KPIs critiques. Ces KPIs permettront de suivre et d'évaluer l'efficacité des tests, la sécurité, la maintenabilité et la fiabilité du code au fur et à mesure de son développement.  

1. **Couverture de code par les tests (Code coverage): 80%**

Ce KPI consiste à mesurer le pourcentage de code source couvert par les tests automatisés. Une couverture élevée permettra de garantir que la majorité du code a été testée, ce qui permet de réduire les risques de bugs en production.

Le seuil minimum que je recommande pour ce KPI est de **80%**.  

Ce pourcentage permet d'assurer que la majorité du code a été testée, mais il laisse également une certaine marge de flexibilité.  
Ce seuil est aligné sur les standards de qualité Sonar Way proposés par SonarQube. Il offre un bon équilibre entre l'effort déployé pour l'implémentation des tests et la détection des problèmes potentiels.

2. **Evaluation de la fiabilité du code (Reliability Rating): A**

Ce KPI consiste à évaluer la fiabilité du code en fonction des bugs détectés. Ils sont classés en fonction de leur sévérité par SonarQube, grâce à une note qui va de A (la meilleure note) à E (la pire note).

Le seuil minimum que je recommande pour ce KPI est **A**.

Cette note permet d'indiquer qu'aucun nouveau bug critique n'a été introduit dans le code ajouté. Cela est essentiel pour maintenir la stabilité de l'application et éviter des régressions qui pourraient nuire à l'expérience utilisateur.

3. **Evaluation de la sécurité du code (Security Rating): A**

Ce KPI consiste à mesurer la sécurité du code en fonction des vulnérabilités détectées par SonarQube. Comme pour la fiabilité, la sécurité est notée de A à E, A étant la meilleure note.

Le seuil minimum que je recommande pour ce KPI est **A**.

Ce score permet de confirmer qu'aucune vulnérabilité critique n'a été introduite dans le code. Assurer un niveau de sécurité élevé est crucial pour protéger les données des utilisateurs et éviter les failles qui pourraient entraîner des dommages significatifs à l'intégrité de l'application et à la confiance des utilisateurs.

4. **Evaluation de la maintenabilité du code (Maintainability Rating): A**

Ce KPI consiste à évaluer la maintenabilité du code en fonction de la dette technique identifiée par SonarQube. La dette technique inclut des aspects tels que la complexité cyclomatique, les duplications de code et la structure générale du code. Les notes vont de A à E, où A est la meilleure note.

Le seuil minimum que je recommande pour ce KPI est **A**.

Ce seuil permet d'indiquer que le code est bien structuré, facile à comprendre et à maintenir. Cela est essentiel pour permettre des évolutions rapides et éviter des coûts élevés de maintenance sur le long terme.

## 5. Analyse des métriques actuelles

### Backend  

#### Résultats d'analyse SonarCloud  

- Couverture: **54.5%**
- Sécurité: **A**
- Fiabilité: **D**
- Maintenabilité: **A**

### Frontend  

#### Résultats d'analyse SonarCloud  

- Couverture: **83.3%**
- Sécurité: **A**
- Fiabilité: **A**
- Maintenabilité: **A**


### Synthèse 

#### Backend:  
La couverture de code du back end est globalement faible avec un pourcentage de **54.5%**. Cela signifie qu'une large portion du code n'est pas testée. Ce qui est bien en dessous du seuil recommandé (80%).
Cette faible couverture expose le backend à un risque élevé de bugs non détectés en production.  
En termes de fiabilité, le backend est noté **D** par SonarCloud, ce qui indique la présence de bugs critiques ou majeurs qui doivent être résolus en priorité pour assurer la stabilité de l'application.  
Cependant, des points positifs sont à noter, notamment une évaluation **A** pour la sécurité et la maintenabilité, ce qui signifie que le code du backend est globalement sécurisé et bien structuré, malgré les problèmes de couverture de tests et de fiabilité.  
Il est crucial d'améliorer la couverture de test et de résoudre les problèmes de fiabilité pour renforcer la robustesse du backend.

#### Frontend:  
La couverture de code du front end est très bonne avec un pourcentage de **83.33%**. Ce qui signifie qu'une large portion du code est testée. Le seuil recommandé (80%) a été dépassé. 
Ces résultats indiquent que le frontend est bien testé, réduisant ainsi les risques de bugs en production. Attention à bien rester au-dessus du seuil lors de l'ajout de nouveauté pour éviter  toute présence de bug. 
L'analyse de SonarCloud nous offre, pour le frontend, des évaluations **A** pour la sécurité, la fiabilité et la maintenabilité. 
Cette analyse nous montre que le code du frontend est bien structuré et sécurisé, mais aussi très fiable, par l'absence de bugs critiques.
Malgré ces bons résultats, il serait quand même préférable d'implémenter des tests sur les fonctions afin d'éviter tout potentiel bug.


## 6. Analyse des retours utilisateurs

Les retours des utilisateurs de BobApp ont fourni des indications précieuses sur les aspects de l'application qui nécessitent une amélioration immédiate.  
Voici un résumé des principaux retours :

```
★☆☆☆☆  
Je mets une étoile car je ne peux pas en mettre zéro ! Impossible de poster une suggestion de blague, le bouton tourne
et fait planter mon navigateur !
```
**Problème:** Mention d'un bug avec un bouton de suggestion de blague, mais cette fonctionnalité n'existe pas dans l'application.  
**Action:** Malgré l'allusion d'une fonctionnalité inexistante par l'utilisateur. Nous pourrions introduire cette fonctionnalité pour répondre aux attentes des utilisateurs.
___
```
★★☆☆☆  
#BobApp j'ai remonté un bug sur le post de vidéo il y a deux semaines et il est encore présent ! Les devs vous faites quoi ????
```
**Problème:** Signalement d'un bug sur le post de vidéo, mais cette fonctionnalité n'existe pas dans l'application.  
**Action:** Comme pour le précédent commentaire, nous pourrions réfléchir à introduire cette fonctionnalité dans les futures versions de l'application pour répondre à la demande des utilisateurs.
___
```
★☆☆☆☆  
Ça fait une semaine que je ne reçois plus rien, j'ai envoyé un email il y a 5 jours mais toujours pas de nouvelles...
```
**Problème:** L'utilisateur ne reçoit pas de réponse à leur demande d'assistance par email.  
**Action:**  Améliorer la réactivité du support en proposant un suivi des demandes sous forme de GitHub Issues pour une gestion plus efficace des problèmes.  
___
```
★★☆☆☆
J'ai supprimé ce site de mes favoris ce matin, dommage, vraiment dommage.
```
**Problème:** L'utilisateur est déçu et a supprimé l'application de ses favoris. Il n'a pas précisé la raison, donc cela est un peu difficile de savoir les problèmes de cet utilisateur.  
**Action:** Augmenter la qualité globale de l'application en corrigeant les bugs existants et en introduisant de nouvelles fonctionnalités attractives pour regagner la confiance des utilisateurs.


## 7. Actions prioritaires

Sur la base des métriques fournies par les couvertures de code, les analyses de SonarCloud et les retours utilisateurs, voici les actions prioritaires à mener pour améliorer BobApp:

1. Résolution des bugs existants

- Corriger les bugs signalés pour améliorer l'expérience utilisateur.
- S'assurer que toutes les fonctionnalités actuelles fonctionnent correctement, et en particulier celles mentionnées par les utilisateurs.

2. Amélioration des réponses aux supports

- Augmenter la réactivité du support en traitant les demandes via GitHub Issues, ce qui facilitera le suivi et la résolution des problèmes remontés.

3. Augmentation de la couverture de code

- Ajouter des tests unitaires et d'intégration pour atteindre le seuil minimum recommandé (80%).  
- Améliorer la couverture des fonctions en testant les fonctionnalités critiques de l'application.  

4. Augmentation de la fiabilité  

- Backend: Corriger le  bug critique relevé par SonarCloud afin de rétablir la note "A" en fiabilité.

5. Maintien du niveau de sécurité

- Continuer à prêter une attention particulière au score de sécurité pour conserver la note "A" au fur et à mesure de l'évolution de l'application.

6. Surveillance de la maintenabilité du code

- Backend: Simplifier le code et améliorer la structure globale du code pour conserver la note "A" en maintenabilité.

7. Prise en compte des retours utilisateurs

- Introduire de nouvelles fonctionnalités, telles que la suggestion de blague et le post de vidéo, pour retenir les utilisateurs et en attirer de nouveaux.  

Ces actions permettront d'améliorer la qualité, la fiabilité, la sécurité et la maintenabilité de l'application BobApp, tout en répondant efficacement aux attentes des utilisateurs.
