# TD2 / TP2 – Utilisation de GitHub et Mise en place d’une CI/CD avec GitHub Actions

## Introduction

Ce document présente le travail réalisé dans le cadre du TD2 (découverte de GitHub) et du TP2 (mise en place d’une intégration continue et d’un début de déploiement via GitHub Actions).  
Objectifs :
- Comprendre Git et GitHub : clonage, commit, push, clé SSH, protection de branche.
- Mettre en place une intégration continue (CI) avec GitHub Actions.
- Construire et publier des images Docker automatiquement.
- Ajouter un contrôle qualité avec SonarCloud.
- Répondre aux questions posées.

---

## 1. TD2 – Découverte de GitHub

### 1.1 Création du compte GitHub

- Création d’un compte GitHub avec l’adresse email scolaire.
- Fork du projet fourni, pour le placer dans son espace GitHub.

### 1.2 Clonage via HTTPS puis via SSH

Clonage via HTTPS :
```bash
git clone https://github.com/<utilisateur>/<projet>.git
````

Clonage via SSH (après génération et ajout d’une clé SSH) :

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/ma-cle-github
# Ajouter ~/.ssh/ma-cle-github.pub dans GitHub > Settings > SSH Keys
git clone git@github.com:<utilisateur>/<projet>.git
```

### 1.3 Commandes Git essentielles

| Commande                  | Description                                           |
| ------------------------- | ----------------------------------------------------- |
| `git status`              | Vérifie les modifications locales                     |
| `git add .`               | Ajoute tous les fichiers modifiés à l’index           |
| `git commit -m "message"` | Sauvegarde les modifications localement               |
| `git push origin master`  | Envoie les commits sur GitHub                         |
| `git pull`                | Récupère les dernières modifications du dépôt distant |

### 1.4 Protection de la branche master

GitHub > Settings > Branches > Add rule :

* Protection de la branche `master`
* Désactivation des push forcés
* Optionnel : obligation de pull request

---

## 2. TP2 – GitHub Actions (CI/CD)

### 2.1 Objectif

Mettre en place un workflow automatisé :

* Lancement à chaque push ou pull request
* Compilation et tests Maven
* Construction d’images Docker
* Publication sur Docker Hub
* Analyse du code (SonarCloud)

### 2.2 Structure à créer

```
.github/
└── workflows/
    └── main.yml
```

### 2.3 Exemple de fichier `.github/workflows/main.yml`

```yaml
name: CI DevOps

on:
  push:
    branches: ["main", "develop"]
  pull_request:

jobs:
  test-backend:
    runs-on: ubuntu-24.04

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup JDK 21
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: "21"

      - name: Build and Test
        run: mvn -B clean verify --file ./simple-api/pom.xml
```

---

### Question 2-1

**Qu’est-ce que Testcontainers et à quoi cela sert-il dans notre projet ?**
**Réponse :**
Testcontainers est une bibliothèque Java qui permet de lancer automatiquement des conteneurs Docker pendant les tests. Dans ce projet, il est utilisé pour démarrer un conteneur PostgreSQL afin de tester l’application avec une vraie base de données sans configuration manuelle.

---

## 3. Construction et publication d’images Docker

### 3.1 Ajout des secrets GitHub

GitHub > Settings > Secrets and variables > Actions > New repository secret :

* `DOCKERHUB_USERNAME`
* `DOCKERHUB_TOKEN`

### Question 2-2

**Pourquoi avons-nous besoin de variables sécurisées (secrets) dans GitHub Actions ?**
**Réponse :**
Les secrets permettent de stocker des informations sensibles (identifiants Docker Hub, token SonarCloud). Ils évitent de mettre ces informations dans le code source ou dans les fichiers YAML visibles publiquement.

---

### 3.2 Ajout du job de build Docker dans le workflow

```yaml
  build-and-push-docker-image:
    needs: test-backend
    runs-on: ubuntu-24.04

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Login to DockerHub
        run: echo "${{ secrets.DOCKERHUB_TOKEN }}" | docker login --username ${{ secrets.DOCKERHUB_USERNAME }} --password-stdin

      - name: Build and Push backend image
        uses: docker/build-push-action@v6
        with:
          context: ./simple-api
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/tp2-backend:latest
          push: ${{ github.ref == 'refs/heads/main' }}
```

### Question 2-3

**Pourquoi avoir mis `needs: test-backend` dans le job de build Docker ?**

**Réponse :**
Cela impose que le job de construction Docker ne démarre que si le job de tests a réussi. Cela évite de construire et publier des images contenant un code qui ne compile pas ou avec des tests en échec.

---

### Question 2-4

**Pourquoi faut-il pousser les images Docker sur Docker Hub ?**
**Réponse :**
Pousser les images sur Docker Hub permet :

* De les rendre accessibles depuis n’importe quelle machine
* De ne pas reconstruire l’image à chaque fois
* De préparer un déploiement sur un serveur, Kubernetes, ou autre
* De partager les images avec d’autres développeurs

---

## 4. Analyse de qualité de code avec SonarCloud

### 4.1 Objectif

Analyser automatiquement le code source pour détecter bugs, failles de sécurité, code dupliqué et mesurer la couverture des tests.

### 4.2 Ajout de SonarCloud dans le workflow

Dans GitHub Secrets : `SONAR_TOKEN`.

```yaml
      - name: SonarCloud Analysis
        run: mvn -B verify sonar:sonar \
          -Dsonar.projectKey=<votre-project-key> \
          -Dsonar.organization=<votre-organisation> \
          -Dsonar.host.url=https://sonarcloud.io \
          -Dsonar.login=${{ secrets.SONAR_TOKEN }} \
          --file ./simple-api/pom.xml
```

---

## Conclusion

À l’issue du TD2/TP2, les éléments suivants ont été réalisés :

* Utilisation complète de Git et GitHub (SSH, commit, push, protection de branche)
* Mise en place d’un pipeline CI avec GitHub Actions
* Exécution automatique des tests Maven
* Construction et publication d’images Docker
* Utilisation de secrets GitHub pour sécuriser les identifiants
* Mise en place d’une analyse qualité avec SonarCloud

```

