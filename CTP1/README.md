# TD1 / TP1 : Découverte de Docker et Mise en place d'une application 3-tiers

## Introduction

Ce document présente ce qui a été réalisé lors du **TD1 (Discover Docker)** et du **TP1 (3-tiers application avec Docker)**. Il inclut :

* Une explication des commandes et concepts utilisés
* Les étapes suivies pour créer les images Docker
* Les réponses **aux questions posées**, reformulées et expliquées **en français**
* La structure du projet et bonnes pratiques

---

## Partie TD1 – Discover Docker

### Objectifs

* Comprendre les bases de Docker (images, conteneurs, client & daemon, etc.)
* Savoir utiliser les commandes : `docker run`, `docker ps`, `docker images`, `docker pull`, `docker stop`, etc.
* Créer un premier conteneur avec une application Flask + Dockerfile

### Étapes principales réalisées

1. Installation de Docker et vérification avec `docker run hello-world`
2. Exécution de conteneurs simples avec Alpine Linux :

   ```bash
   docker run alpine ls -l
   docker run alpine echo "hello from alpine"
   docker run -it alpine /bin/sh
   ```
3. Utilisation de `docker ps`, `docker ps -a`, `docker images`
4. Déploiement d’un site statique avec `dockersamples/static-site`
5. Création d’une application Flask avec **Dockerfile** complet

### Questions du TD1

#### **Question 1 : Quelle est la différence entre une image et un conteneur ?**

**Réponse :**
Une **image Docker** est un modèle figé (lecture seule) contenant tout le nécessaire pour exécuter une application (code, dépendances, configuration...). Un **conteneur Docker** est une instance **en cours d'exécution** d’une image, avec son propre système de fichiers isolé et ses processus.

#### **Question 2 : Que fait exactement la commande `docker run` ?**

**Réponse :**

1. Docker vérifie si l’image est disponible localement, sinon il la télécharge.
2. Il crée un conteneur basé sur cette image.
3. Il exécute la commande fournie dans le conteneur.
4. Il affiche la sortie dans le terminal (mode interactif ou non).

---

## Partie TP1 – Application 3-Tiers

### Objectif final

Mettre en place une **application web 3-couches** :

* **Base de données** (PostgreSQL + Adminer)
* **Backend API** (Spring Boot / Java)
* **Serveur HTTP** (Apache ou Nginx en reverse proxy)
* Orchestration avec **docker-compose**

## Questions du TP1 et Réponses

### **Question 1-1 :**

**Pourquoi vaut-il mieux passer les variables d’environnement avec `-e` au moment du `docker run` plutôt que directement dans le Dockerfile ?**

**Réponse :**
Il est préférable de passer ces variables avec `-e` ou via un fichier `.env` car :

* Cela évite de stocker des mots de passe en clair dans le Dockerfile (meilleure sécurité)
* Le Dockerfile peut être versionné sans exposer d’informations sensibles
* Cela rend l’image générique et réutilisable dans plusieurs environnements (dev, test, prod)

### **Question 1-2 :**

**Pourquoi a-t-on besoin d’un volume pour PostgreSQL ?**

**Réponse :**
Un volume permet de **persister les données** en dehors du conteneur. Sans volume, si le conteneur est supprimé ou redémarré, toutes les données de la base seraient perdues. Avec un volume, les données sont stockées sur la machine hôte de manière durable.

### **Question 1-3 :**

**Documentez les éléments essentiels de votre conteneur de base de données : commandes et Dockerfile.**

Dockerfile minimal :

```
FROM postgres:17.2-alpine

ENV POSTGRES_DB=db \
    POSTGRES_USER=usr \
    POSTGRES_PASSWORD=pwd
```

Ici, l’image officielle PostgreSQL est utilisée, et les variables d’environnement définissent la base par défaut, l’utilisateur et son mot de passe.

Commandes essentielles :

1. Construire l’image :

```bash
docker build -t my-postgres .
```

2. Créer un réseau Docker (pour communiquer avec Adminer ou API) :

```bash
docker network create app-network
```

3. Lancer le conteneur avec un nom, un réseau, un volume (persistance) et des variables :

```bash
docker run -d \
  --name my-postgres \
  --network app-network \
  -e POSTGRES_DB=db \
  -e POSTGRES_USER=usr \
  -e POSTGRES_PASSWORD=pwd \
  -v pgdata:/var/lib/postgresql/data \
  postgres:17.2-alpine
```

4. Lancer Adminer pour tester la base :

```bash
docker run -d \
  --name adminer \
  --network app-network \
  -p 8090:8080 \
  adminer
```

### **Question 1-4 :**

**Pourquoi utilise-t-on une construction en plusieurs étapes ? Expliquez chaque étape du Dockerfile.**

**Réponse :**
La construction en plusieurs étapes permet de réaliser des images Docker plus légères et optimisées. Il consiste à utiliser plusieurs blocs `FROM` dans un Dockerfile :

* **Première étape (builder)** : contient tout ce qui permet de compiler l'application (JDK, Maven, dépendances...). Le but est de produire les artefacts nécessaires (ex : `.jar`).
* **Deuxième étape (runtime)** : utilise seulement ce qui est nécessaire pour exécuter l’application (JRE uniquement). On y copie uniquement le fichier `.jar` ou `.class` compilé depuis l’étape précédente.

**Avantages :**
 Image finale plus légère
 Pas besoin d’avoir Maven ou JDK dans l’image de production
 Plus sécurisé

### **Question 1-5 :**

**Pourquoi avons-nous besoin d’un reverse proxy (proxy inverse) ?**

**Réponse :**
Un reverse proxy est nécessaire pour :

* Servir d’intermédiaire entre les utilisateurs et l’application backend
* Centraliser les accès (URL unique : `http://localhost`)
* Cacher les ports internes (`8080`, `5432`, etc.)
* Permettre plus tard l’ajout de HTTPS, équilibrage de charge, cache, etc.

Exemple : Apache/Nginx reçoit la requête puis la redirige vers le backend Spring Boot.

### **Question 1-6 :**

**Pourquoi docker-compose est-il si important ?**

**Réponse :**
`docker-compose` permet de :

* Définir **plusieurs containers comme un seul ensemble logique** (backend + bdd + httpd)
* Lancer, arrêter, reconstruire tous les services avec une seule commande
* Gérer les networks, les volumes, les variables d’environnement
* Faciliter le déploiement en équipe ou sur serveur

### **Question 1-7 :**

**Commandes Docker Compose importantes :**

| Commande                 | Description                                                       |
| ------------------------ | ----------------------------------------------------------------- |
| `docker-compose up`      | Lance les containers définis dans le fichier `docker-compose.yml` |
| `docker-compose up -d`   | Lance en arrière-plan                                             |
| `docker-compose down`    | Stoppe et supprime les containers, networks, mais pas les volumes |
| `docker-compose build`   | (Re)construit les images                                          |
| `docker-compose ps`      | Liste les services en cours                                       |
| `docker-compose logs -f` | Affiche les logs en temps réel                                    |

### **Question 1-8 :**

**Documentez votre fichier docker-compose.yml.**

```
version: "3.9"

services:
  database:
    image: postgres:17.2-alpine
    container_name: my-postgres
    environment:
      POSTGRES_DB: db
      POSTGRES_USER: usr
      POSTGRES_PASSWORD: pwd
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
      - app-network

  backend:
    build: ./simple-api
    container_name: backend-api
    depends_on:
      - database
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://my-postgres:5432/db
      SPRING_DATASOURCE_USERNAME: usr
      SPRING_DATASOURCE_PASSWORD: pwd
    networks:
      - app-network

  httpd:
    build: ./http-server
    container_name: http-server
    depends_on:
      - backend
    ports:
      - "80:80"
    networks:
      - app-network

networks:
  app-network:

volumes:
  pgdata:
```

Explication des parties clés :

| Élément        | Rôle                                                             |
| -------------- | ---------------------------------------------------------------- |
| `services:`    | Liste des conteneurs à lancer (base de données, backend, httpd). |
| `depends_on:`  | Définit l’ordre de démarrage (ex : backend après database).      |
| `environment:` | Variables d’environnement (accès à PostgreSQL).                  |
| `volumes:`     | Persistance des données (`pgdata`).                              |
| `networks:`    | Tous les conteneurs communiquent via `app-network`.              |
| `ports:`       | Seul Apache (HTTP) expose un port vers l’hôte (`80:80`).         |
| `build:`       | Indique le chemin où se trouve le Dockerfile du service.         |


Pourquoi c’est important ?

Le fichier docker-compose.yml permet de lancer toute l’architecture avec une seule commande :

```bash
docker-compose up -d
```

Il garantit une architecture propre, isolée, reproductible.

### **Question 1-9 :**

**Quelles commandes pour publier nos images sur Docker Hub ?**

**Réponse :**

1. Se connecter à Docker Hub

   ```bash
   docker login
   ```
2. Taguer l’image :

   ```bash
   docker tag my-image USERNAME/my-image:1.0
   ```
3. Pousser l’image :

   ```bash
   docker push USERNAME/my-image:1.0
   ```
4. L’image est désormais accessible publiquement ou privée sur Docker Hub.

### **Question 1-10 :**

**Pourquoi mettre nos images dans un dépôt en ligne ?**

**Réponse :**

* Permet de **partager facilement** les images avec d’autres machines ou collaborateurs
* Garantit que l’image est **stockée de façon centralisée** et sauvegardée
* Simplifie le **déploiement CI/CD**
* Évite d’avoir à reconstruire l’image sur chaque machine

