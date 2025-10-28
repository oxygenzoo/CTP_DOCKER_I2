# TD3 – Discover Ansible

## Pré-requis
Ansible installé dans WSL2 (Ubuntu 22.04).  
Accès SSH configuré vers le serveur : enzo.guignabert.takima.cloud

## Configuration SSH
- Copie de la clé privée dans `~/.ssh/takima`
- Droits sécurisés sur la clé avec `chmod 400`
- Test SSH réussi vers l’utilisateur `admin`

## Inventaire Ansible
Serveur déclaré dans `/etc/ansible/hosts` puis dans `ansible/inventory` :
enzo.guignabert.takima.cloud

## Test Ansible Ping
Commande :
ansible all -m ping --private-key=~/.ssh/takima -u admin

Résultat : Réponse `pong`

Ce test montre qu’Ansible :
- voit le serveur
- peut s’y connecter en SSH avec la clé privée
- peut exécuter des commandes à distance

## Installation d’Apache
Commande :
ansible all -m apt -a "name=apache2 state=present update_cache=true" --private-key=~/.ssh/takima -u admin --become

Apache installé avec succès

## Déploiement d’une page HTML
Commande :
ansible all -m shell -a 'echo "<html><h1>Hello World</h1></html>" > /var/www/html/index.html' --private-key=~/.ssh/takima -u admin --become

Accès navigateur :
http://enzo.guignabert.takima.cloud

Page affichée : “Hello World”

---

## Playbook Ansible

Un playbook `site.yml` a été créé pour automatiser :
- installation Apache
- déploiement de la page web
- démarrage du service

Commande de lancement :
ansible-playbook -i ansible/inventory ansible/site.yml --private-key=~/.ssh/takima -u admin


---

## Conclusion
Ce TD m’a permis de découvrir les bases d’Ansible :
- configuration SSH
- gestion d’un inventaire d’hôtes
- exécution de modules à distance
- installation et configuration d’un serveur web
- création d’un playbook automatisé

Première mise en production réussie