# TP partie 1

> **1-1 Présentez les composants clés de votre conteneur de base de données, y compris les commandes et le Dockerfile.**

## **Fichier Dockerfile**

```Dockerfile
# Sélection de l'image de base
FROM postgres:14.1-alpine
```

```Dockerfile
# Définition des variables d'environnement pour la configuration
ENV POSTGRES_DB=db \
    POSTGRES_USER=usr \
    POSTGRES_PASSWORD=pwd
```

```Dockerfile
# Importation des scripts nécessaires à l'initialisation
COPY ./init-db-scripts/ /docker-entrypoint-initdb.d/
```

```Dockerfile
# Mise à disposition du port pour les connexions
EXPOSE 5432
```

```Dockerfile
# Ajout de packages utiles, réglage du fuseau horaire et nettoyage post-installation
RUN apk add --no-cache \
    bash \
    vim \
    tzdata \
    && cp /usr/share/zoneinfo/Europe/Paris /etc/localtime \
    && echo "Europe/Paris" > /etc/timezone \
    && apk del tzdata
```

```Dockerfile
# Ajustement des permissions et création d'un lien symbolique pour le point d'entrée
RUN chmod 777 /usr/local/bin/docker-entrypoint.sh \
    && ln -s /usr/local/bin/docker-entrypoint.sh /
```

```Dockerfile
# Création d'un volume pour la persistance des données
VOLUME ["/var/lib/postgresql/data"]
```

## **Script run.sh**

### PostgreSQL

```bash
# Mise en place d'un réseau Docker personnalisé
docker network create my-network > /dev/null;
```

```bash
# Nettoyage de l'instance PostgreSQL précédente si elle existe
docker stop postgres > /dev/null;
docker rm postgres > /dev/null;
```

```bash
# Compilation de l'image PostgreSQL à partir du Dockerfile
docker build -t postgres . --no-cache > /dev/null
```

```bash
# Lancement du conteneur PostgreSQL en arrière-plan, accessible via le port 5432
docker run -d -p 5432:5432 --name postgres postgres > /dev/null
```

### Adminer

```bash
# Suppression de l'instance Adminer précédente si nécessaire
docker stop adminer > /dev/null;
docker rm adminer > /dev/null;
```

```bash
# Démarrage d'Adminer en arrière-plan, connecté au réseau PostgreSQL, sur le port 9090
docker run -d --link postgres:db -p 9090:8080 --name adminer adminer > /dev/null
```

> **1-2 Pourquoi la construction par étapes est-elle nécessaire ? Décortiquez chaque segment de ce Dockerfile.**

La construction en étapes est cruciale pour minimiser la taille de l'image Docker finale, en séparant les étapes de construction et d'exécution de l'application. Cela permet d'éliminer les dépendances inutiles à l'exécution, réduisant ainsi l'empreinte globale de l'image.

> **1-3 Expliquez les commandes essentielles de docker-compose.**

`docker-compose up` : Lance les services définis\
`docker-compose down` : Arrête et supprime les ressources associées\
`docker-compose ps` : Liste les services et leur état\
`docker-compose logs` : Affiche les logs des services\
`docker-compose exec` : Exécute une commande dans un service\
`docker-compose build` : Construit ou reconstruit les services\
`docker-compose pull` : Récupère les images des services\
`docker-compose restart` : Redémarre tous les services

> **1-4 Expliquez la structure de votre fichier docker-compose.**

- `version: '3.7'` : Détermine la version de Docker Compose utilisée.
- `services` : Définit les différents services du projet.
  - `backend`
    - `build: backend/.` : Construit l'image du backend depuis le dossier spécifié.
    - `container_name: backend` : Attribue un nom au conteneur.
    - `networks: my-network` : Connecte le conteneur au réseau défini.
    - `depends_on: database, httpd` : Assure que le backend démarre après la base de données et le serveur web.
  - `database`
    - `ports: "5432:5432"` : Rend le port 5432 du conteneur accessible

 depuis l'hôte.
    - `build: db/.` : Construit l'image de la base de données.
    - `container_name: postgres` : Nomme le conteneur de la base de données.
    - `networks: my-network` : Connecte au réseau spécifié.
  - `httpd`
    - `ports: "80:80"` : Expose le port 80 du conteneur sur le port 80 de l'hôte.
    - `build: webserver/.` : Construit l'image du serveur web.
    - `container_name: webserver` : Nomme le conteneur du serveur web.
    - `networks: my-network` : L'ajoute au réseau défini.
  - `networks` : Configure le réseau personnalisé.
    - `my-network` : Nom du réseau qui relie les services.

> **1-5 Documentez la procédure de publication de vos images sur Docker Hub.**

`docker login -u <usr>` suivi du mot de passe : Authentifie sur Docker Hub\
`docker compose build` : Construit les images selon le fichier docker-compose\
`docker image ls` : Liste les images disponibles localement\
`docker tag <img_name>:<version> <usr>/<img_name>:<version>` : Assigne un tag à l'image pour Docker Hub\
`docker push <usr>/<img_name>:<version>` : Publie l'image sur Docker Hub\

# TP partie 2

> **2-1 Qu'est-ce que Testcontainers ?**

Testcontainers est une bibliothèque Java facilitant les tests d'intégration en utilisant des conteneurs Docker éphémères pour les bases de données, les navigateurs web, et d'autres services, rendant les tests plus fiables et rapides, en particulier dans un environnement d'intégration continue.

**Avantages :**

```
* Simplification de la gestion des conteneurs
* Amélioration de l'efficacité des tests
* Réduction du temps de développement
* Accroissement de la fiabilité des tests en simulant des environnements réels
```

> **2-2 Expliquez la configuration de vos workflows GitHub Actions.**

**Définition du nom du processus**

```yml
name: CI devops 2023
```

---

**Déclenchement du workflow lors de modifications sur la branche principale**

```yml
on:
  push:
    branches: main
```

---

**Création d'une tâche nommée "Test de l'application Spring" sur `Ubuntu 22.04` et détail des étapes**

```yml
jobs:
  test-backend:
    name: "Test de l'application Spring"
    runs-on: ubuntu-22.04
    steps:
```

---

**Récupération du code source depuis le dépôt GitHub**

```yml
- uses: actions/checkout@v2.5.0
```

---

**Configuration de `JDK 17` avec `Temurin` pour l'environnement de build**

```yml
- name: Set up JDK 17
  uses: actions/setup-java@v3
  with:
    java-version: "17"
    distribution: "temurin"
```

---

**Spécification du répertoire de travail et exécution de Maven pour tester et construire le projet**

```yml
- name: Build and test with Maven
  working-directory: ./simple-api-student-main
  run: mvn clean verify
```

---

**Définition d'une tâche pour construire et publier l'image Docker, dépendant des résultats des tests backend**

```yml
build-and-push-docker-image:
  name: "Build and push Docker image"
  needs: test-backend
  runs-on: ubuntu-22.04
  steps:
```

---

**Récupération du code source pour la construction de l'image Docker**

```yml
- name: Checkout code
  uses: actions/checkout@v2.5.0
```

---

**Authentification sur Docker Hub pour publier l'image**

```yml
- name: Login to DockerHub
  run: docker login -u ${{ secrets.DOCKERHUB_USERNAME }} -p ${{ secrets.DOCKERHUB_TOKEN }}
```

---

**Construction et publication de l'image Docker pour le backend, la base de données et le serveur HTTP**

```yml
- name: Build image and push backend
  uses: docker/build-push-action@v3
  with:
    context: ./simple-api-student
    tags: ${{secrets.DOCKERHUB_USERNAME}}/devops-backend:

1.0.0
    push: ${{ github.ref == 'refs/heads/main' }}
```

```yml
- name: Build image and push database
  uses: docker/build-push-action@v3
  with:
    context: ./db
    tags: ${{secrets.DOCKERHUB_USERNAME}}/devops-database:1.0.0
    push: ${{ github.ref == 'refs/heads/main' }}
```

```yml
- name: Build image and push httpd
  uses: docker/build-push-action@v3
  with:
    context: ./html
    tags: ${{secrets.DOCKERHUB_USERNAME}}/devops-httpd:1.0.0
    push: ${{ github.ref == 'refs/heads/main' }}
```

---

> **2-3 Expliquez la configuration de votre portail qualité.**

Nos critères de qualité exigent une note minimale de A pour chaque métrique. La couverture des tests doit être au moins de 80%, et le taux de duplication doit rester en dessous de 3%. Un échec à respecter ces standards entraînera l'échec de la phase de contrôle qualité dans notre intégration continue.

# TP partie 3

> **3-1 Décrivez votre inventaire et les commandes fondamentales.**

Inventaire (`setup.yml`) :

```yml
all:
  vars:
    ansible_user: centos
    ansible_ssh_private_key_file: .ansible_key
  children:
    prod:
      hosts: centos@chatelet.ferrety.matias.takima.cloud
```

Commandes fondamentales :

- Vérification de la connectivité :

  ```
  ansible all -i inventories/setup.yml -m ping
  ```

  Cette commande teste la connectivité avec tous les hôtes listés dans l'inventaire via le module ping d'Ansible.

- Collecte des informations sur la distribution OS :

  ```
  ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"
  ```

  Récupère et filtre les informations concernant la distribution du système d'exploitation des hôtes ciblés.

- Gestion des paquets avec Yum - Suppression d'Apache HTTP Server :

  ```
  ansible all -i inventories/setup.yml -m yum -a "name=httpd state=absent"
  ```

  Utilise le module yum pour supprimer le paquet httpd de tous les hôtes listés dans l'inventaire.

> **3-2 Décrivez votre playbook.**

`playbook.yml` :

```yml
- hosts: all
  gather_facts: false
  become: true

  roles:
    - docker

  tasks:
    - name: Test connection
      ping:
```

Ce playbook configure les hôtes cibles pour l'utilisation de Docker :

- `hosts: all` : Cible tous les hôtes listés dans l'inventaire.
- `gather_facts: false` : Désactive la collecte automatique d'informations sur les hôtes.
- `become: true` : Active les privilèges de super-utilisateur.
- `roles`: Applique le rôle docker pour installer Docker et ses dépendances.
- `tasks`: Liste des tâches à exécuter, ici un simple test de ping pour vérifier la connectivité.

---

`roles/docker/tasks/main.yml` :

```yml
# Tâches pour le rôle Docker
- name: Install device-mapper-persistent-data
  yum:
    name: device-mapper-persistent-data
    state: latest

- name: Install lvm2
  yum:
    name: lvm2
    state: latest

- name: Add Docker repo
  command:
    cmd: sudo yum-config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo

- name: Install Docker
  yum:
    name: docker-ce
    state: present

- name: Install Python 3
  yum:
    name: python3
    state: present

- name: Install Docker module for Python 3
  pip:
    name: docker
    executable: pip3
  vars:
    ansible_python_interpreter: /usr/bin/python3

- name: Ensure Docker service is running
  service:
    name: docker
    state: started
  tags: docker
```

**Installation des dépendances** : Installe les paquets nécessaires pour Docker.
**Ajout du dépôt Docker** : Ajoute le dépôt officiel Docker à la liste des sources de yum.
**Installation

 de Docker** : Installe la dernière version stable de Docker.
**Configuration de Python 3 et du module Docker** : Installe Python 3 et le module Docker pour permettre la gestion des conteneurs via des scripts Python.
**Démarrage du service Docker** : S'assure que le service Docker est actif et démarré.

> **3-3 Expliquez la configuration des tâches `docker_container`.**

### `app` :

#### Tâche 1 : Arrêt et suppression du conteneur

- Utilise `docker_container` pour gérer le cycle de vie du conteneur.
- Supprime le conteneur s'il existe, en ignorant les erreurs.

#### Tâche 2 : Déploiement du conteneur

- Télécharge l'image si nécessaire et démarre un nouveau conteneur avec les configurations spécifiées, notamment le réseau et les ports.

#### Tâche 3 : Confirmation du déploiement

- Affiche un message confirmant le succès du déploiement.

---

### `database` :

- **Arrêt et suppression du conteneur** : Identique à l'application, mais pour le conteneur de la base de données.
- **Déploiement du conteneur de la base de données** : Configure et démarre le conteneur de la base de données.
- **Confirmation** : Affiche un message de succès.

### `docker` :

- **Installation des dépendances** : Prépare le système pour Docker en installant les paquets nécessaires.
- **Configuration de Docker** : Ajoute le dépôt Docker et installe Docker.
- **Démarrage de Docker** : Assure que le service Docker est actif.

### `frontend` :

- Répète les étapes similaires à `app` pour le conteneur frontal, en gérant son cycle de vie et en configurant son déploiement.

### `network` :

- **Création de réseau** : Établit un réseau Docker pour la communication entre les conteneurs.

### `proxy` :

- Gère le cycle de vie du conteneur proxy, similaire à `app` et `database`, en s'assurant de sa configuration et de son déploiement corrects.

### `setup.yml` :

Définit les variables globales et les configurations pour les hôtes, incluant les informations d'authentification, les configurations réseau et les détails spécifiques aux conteneurs comme les noms d'utilisateur et les mots de passe de la base de données.