# Projet – TP/TD du module **Docker Containers** (EFREI)

Ce dépôt regroupe l’ensemble des travaux réalisés dans le cadre du module **Docker Containers**, enseigné à **l’EFREI Paris**.
Tous les travaux pratiques (**TP**) et dirigés (**TD**) ont été **entièrement réalisés par Enzo Guignabert**.

L’objectif de ce projet est de mettre en œuvre progressivement des outils modernes utilisés dans le DevOps : **Docker, Docker Compose, GitHub, GitHub Actions, Ansible, CI/CD, SonarCloud**, etc.

Chaque TP/TD dispose de son propre répertoire ainsi que d’un **README détaillé**, contenant :

* les objectifs,
* les étapes réalisées,
* les configurations (Dockerfile, docker-compose, Ansible, workflows GitHub Actions…),
* **les réponses complètes aux questions posées dans le sujet**.

Ce présent README n’a pas vocation à contenir ces détails techniques, mais à servir de **vue d’ensemble du module**.

---

## Organisation du dépôt

| Dossier                      | Contenu                          | Description                                                                                                            |
| ---------------------------- | -------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `TD1-TP1/`                   | Docker & Application 3-tiers     | Introduction à Docker, création d’images, conteneurs, volumes, docker-compose, architecture 3-tiers.                   |
| `TD2-TP2/`                   | Git & GitHub Actions             | Découverte de GitHub, bonnes pratiques Git, mise en place d'une CI (tests Maven, build).                               |
| `TD3-TP3/`                   | Ansible & Déploiement automatisé, CI/CD avancée & SonarCloud  | Introduction à Ansible, inventaires, playbooks, rôles, installation automatisée de Docker, déploiement des conteneurs. Analyse de code avec SonarCloud, publication sur Docker Hub, déploiement continu (CD).  |

Chaque dossier contient :

* Un **README spécifique** (instructions détaillées, schémas, réponses aux questions),
* Le **code, les fichiers YAML (docker-compose, workflows GitHub Actions, inventaires Ansible)**,
* Les **Dockerfiles** et scripts nécessaires.

---

## Compétences abordées

Ce projet permet de maîtriser les notions suivantes :

### Docker et Conteneurisation

* Images, conteneurs, registres Docker Hub
* Volumes, réseaux Docker
* Dockerfile et multi-stage build
* Architecture 3-tiers (Base de données, API, Reverse proxy)

### Git & GitHub

* Versioning, branches, commits, pull/push
* Clonage via HTTPS / SSH
* Protection des branches (main/master)
* Gestion des secrets GitHub

### Intégration Continue (CI)

* GitHub Actions (.github/workflows)
* Compilation Maven + Tests automatiques
* Testcontainers (PostgreSQL en conteneur pendant les tests)

### Déploiement Continu (CD)

* Build et push automatique d’images sur Docker Hub
* Variables sécurisées (GitHub Secrets)
* Déploiement automatique conditionnel

### Ansible (Automatisation des serveurs)

* Inventaires YAML
* Connexion SSH et exécution distante
* Playbooks et rôles
* Installation de Docker et exécution de conteneurs sur serveur distant

---

## Auteur

**Enzo Guignabert**
Étudiant à l’**EFREI Paris**
Module : **Docker Containers (DevOps)**

---

## Pour plus d’informations

Chaque TD/TP dispose de son propre fichier **README.md** comprenant :

* Les étapes détaillées,
* Les fichiers de configuration,
* Et **les réponses complètes à chaque question du sujet**.
