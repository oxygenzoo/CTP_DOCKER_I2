# TD3 / TP3 – Déploiement avec Ansible et Automatisation CI/CD

## Introduction

Ce document présente le travail effectué dans le cadre du **TD3 (Découverte d’Ansible)** et du **TP3 (Déploiement automatisé avec Ansible)**.
Objectifs principaux :

* Découvrir Ansible et sa logique déclarative.
* Se connecter à un serveur distant via SSH et exécuter des commandes avec Ansible.
* Créer un inventaire, des playbooks et des rôles.
* Installer Docker, exécuter des conteneurs (base de données, API, proxy).
* Automatiser le déploiement via GitHub Actions (CD).
* Répondre aux questions demandées.

---

## 1. TD3 – Découverte d’Ansible

### 1.1 Installation et vérification d’Ansible

```
ansible --version
```

### 1.2 Connexion SSH au serveur distant

```
chmod 400 ~/.ssh/id_rsa
ssh -i ~/.ssh/id_rsa admin@<nom-du-serveur>
```

### 1.3 Test de connexion avec Ansible (ping)

Ajout du serveur dans `/etc/ansible/hosts`, puis :

```
ansible all -m ping --private-key=~/.ssh/id_rsa -u admin
```

Résultat attendu : `pong`

### 1.4 Installation Apache avec Ansible

```
ansible all -m apt -a "name=apache2 state=present" -u admin --private-key=~/.ssh/id_rsa --become
```

### 1.5 Création d’une page web de test

```
ansible all -m shell -a 'echo "<h1>Hello World</h1>" > /var/www/html/index.html' \
-u admin --private-key=~/.ssh/id_rsa --become
```

---

## 2. TP3 – Ansible : Inventaire, Playbooks, Déploiement

### 2.1 Structure Ansible dans le projet

```
ansible/
├── inventories/
│   └── setup.yml
├── playbook.yml
└── roles/
```

### 2.2 Fichier d’inventaire `ansible/inventories/setup.yml`

```
all:
  vars:
    ansible_user: admin
    ansible_ssh_private_key_file: ~/.ssh/id_rsa
  children:
    prod:
      hosts:
        <nom-du-serveur>:
```

### 2.3 Test de l’inventaire

```
ansible all -i inventories/setup.yml -m ping
```

---

## Question 3-1 : Documenter l’inventaire et les commandes de base

**Inventaire :**

* Fichier YAML regroupant les serveurs et leurs variables (utilisateur, clé SSH, groupes).
* Permet à Ansible de savoir où exécuter les commandes.

**Commandes de base utilisées :**

| Commande                                                 | Rôle                        |
| -------------------------------------------------------- | --------------------------- |
| `ansible all -m ping`                                    | Vérifie la connexion SSH    |
| `ansible all -m apt -a "name=apache2 state=present"`     | Installe un paquet          |
| `ansible all -m shell -a "commande"`                     | Exécute une commande shell  |
| `ansible-playbook -i inventories/setup.yml playbook.yml` | Exécute un playbook complet |

---

## 2.4 Premier playbook : test de connexion

```
- hosts: all
  gather_facts: false
  become: true

  tasks:
    - name: Test de connexion
      ping:
```

Exécution :

```
ansible-playbook -i inventories/setup.yml playbook.yml
```

---

## Question 3-2 : Documenter le playbook

Un playbook :

* Décrit l’état souhaité du serveur.
* Contient des tâches organisées en YAML.
* Peut exécuter des rôles.

Exemple :

```
- hosts: all
  become: true
  gather_facts: true

  tasks:
    - name: Installer Docker
      apt:
        name: docker-ce
        state: present
```

---

## 2.5 Installation de Docker via un rôle

Création du rôle :

```
ansible-galaxy init roles/docker
```

Déplacement des tâches d’installation dans `roles/docker/tasks/main.yml`.

---

## 2.6 Déploiement de l’application Dockerisée

Exemple de tâche pour lancer un conteneur :

```
- name: Lancer la base de données
  docker_container:
    name: database
    image: <utilisateur>/tp-devops-database:latest
    env:
      POSTGRES_DB: db
      POSTGRES_USER: usr
      POSTGRES_PASSWORD: pwd
    state: started
    restart_policy: always
```

---

## Question 3-3 : Documenter la configuration `docker_container`

* `name` : nom du conteneur
* `image` : image Docker utilisée (provenant de Docker Hub)
* `env` : variables d’environnement (mot de passe, utilisateur, base…)
* `state: started` : garantit que le conteneur est lancé
* `restart_policy: always` : redémarrage automatique en cas de crash

---

## 3. Automatisation du déploiement (CD) dans GitHub Actions

### 3.1 Intégration dans `.github/workflows/main.yml`

Ajout après push d’images :

```
- name: Déploiement distant avec Ansible
  run: ansible-playbook -i ansible/inventories/setup.yml ansible/playbook.yml
```

---

## Question finale

**Est-il sûr de déployer automatiquement chaque nouvelle image ? Que faire pour sécuriser ?**

**Réponse :**
Non, ce n’est pas totalement sûr. Déployer directement chaque image publiée peut introduire du code non testé ou non validé en production.
Pour sécuriser :

* Ajouter une validation manuelle (approval) dans GitHub Actions.
* Déployer uniquement les images taguées `release` ou `main`.
* Utiliser des environnements protégés (`environments` + `protection rules`).
* Ajouter des tests end-to-end avant déploiement.

---

## Conclusion

Ce TP3 a permis de :

* Comprendre l’utilisation d’Ansible pour automatiser un serveur.
* Créer un inventaire, des playbooks et des rôles.
* Installer Docker et déployer une application complète (API, DB, Proxy).
* Automatiser le déploiement via GitHub Actions et Docker Hub.
* Réfléchir à la sécurité du déploiement continu.
