Compte-Rendu TPs DevOps - Grégory ADELL :

TP 1 :

1-1 Why should we run the container with a flag -e to give the environment variables?

Pour éviter d'avoir les login et mot de passes écris en clair dans un fichier texte facilement lisible comme le Dockerfile. ça permet de renforcer la sécurité.
Exemple avec les identifiants indiqués dans la commande via "-e" :
    docker run -d --name postgres-container --network app-network -e POSTGRES_DB=db -e POSTGRES_USER=user -e POSTGRES_PASSWORD=pwd -p 5432:5432 postgres:14.1-alpine 

--------------------------------------------------------------

1-2 Why do we need a volume to be attached to our postgres container?

On a besoin d'un volume pour éviter de perdre des données. Les volume permet de stocker des données persistantes en dehors du cycle de vie du conteneur. ça assure leur conservation même après la duppression et el redéploiement du conteneur.

--------------------------------------------------------------

1-3 Document your database container essentials: commands and Dockerfile.

FROM postgres:14.1-alpine

ENV POSTGRES_DB=db \
    POSTGRES_USER=user \
    POSTGRES_PASSWORD=pwd

# Copier les scripts SQL d'initialisation
COPY 01-CreateScheme.sql /docker-entrypoint-initdb.d/
COPY 02-InsertData.sql /docker-entrypoint-initdb.d/


Commandes utilisées :

1- Créer un réseau Docker :

docker network create app-network



2- Construire l’image PostgreSQL :

docker build -t postgres .



3- Exécuter le conteneur avec persistance et connexion réseau :

docker run -d --name postgres-db --network app-network -v pg_data:/var/lib/postgresql/data -e POSTGRES_DB=db -e POSTGRES_USER=user -e POSTGRES_PASSWORD=pwd postgres



4- Lancer Adminer pour gérer la base:

docker run -p 8090:8080 --network app-network --name=adminer -d adminer

--------------------------------------------------------------

1-4 Why do we need a multistage build? And explain each step of this dockerfile.

Une construction multi-étapes permet de réduire la taille des images Docker en séparant l’étape de compilation et l’étape d’exécution.
Explication du Dockerfile multi-étapes :

# Étape 1 : Compilation de l’application Java avec Maven
# Build
FROM maven:3.9.9-amazoncorretto-21 AS myapp-build
ENV MYAPP_HOME=/opt/myapp 
WORKDIR $MYAPP_HOME
COPY pom.xml .
COPY src ./src
RUN mvn package -DskipTests

# Étape 2 : Création de l’image finale avec seulement le JRE
# Run
FROM amazoncorretto:21
ENV MYAPP_HOME=/opt/myapp 
WORKDIR $MYAPP_HOME
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

ENTRYPOINT ["java", "-jar", "myapp.jar"]

Pourquoi ?

    L’étape build utilise Maven + JDK pour compiler l’application.
    L’étape run utilise uniquement un JRE, ce qui réduit considérablement la taille de l’image et accélère son exécution.

--------------------------------------------------------------

1-5 Why do we need a reverse proxy?

Le reverse proxy permet d’optimiser la gestion du trafic (Il redirige les requêtes vers le bon service (backend)). Il permet aussi de masquer les services internes. Les utilisateurs n’accèdent donc qu’au serveur proxy et non pas directement aux autres conteneurs. Aussi, le reverse proxy permet de gérer les certificats SSL. Par exemple Apache ou Nginx peut gérer HTTPS et faire du load balancing.

--------------------------------------------------------------

1-6 Why is docker-compose so important?

Docker Compose permet de :
- Définir et gérer plusieurs conteneurs dans un seul fichier docker-compose.yml.
- Automatiser le déploiement en une seule commande (docker-compose up).
- Gérer les dépendances (ex: exécuter PostgreSQL avant le backend).
- Simplifier la configuration en évitant d’avoir à taper plusieurs commandes docker run.

--------------------------------------------------------------

1-7 Document docker-compose most important commands. 

Démarrer les conteneurs :
docker-compose up -d

Arrêter les conteneurs :
docker-compose down

Recréer les conteneurs après un changement :
docker-compose up --build -d

Vérifier les logs :
docker-compose logs

Lister les services :
docker-compose ps

--------------------------------------------------------------

1-8 Document your docker-compose file.



--------------------------------------------------------------

1-9 Document your publication commands and published images in dockerhub.

Connexion à Docker Hub :
docker login

Taguer une image :
docker tag my-database USERNAME/my-database:1.0

Pousser une image sur Docker Hub :
docker push USERNAME/my-database:1.0

Images publiées :
- USERNAME/my-database:1.0 : Image PostgreSQL avec base préconfigurée.
- USERNAME/my-backend:1.0 : Application backend Spring Boot.
- USERNAME/my-httpd:1.0 : Serveur HTTP Apache configuré comme reverse proxy.

--------------------------------------------------------------

1-10 Why do we put our images into an online repo?

Pour faciliter le partage, pour le déploiement automatisé ou encore pour le "Versioning" (permet de gérer les différentes version d'images et par exemple revenir facilement à une version antiérieure).

--------------------------------------------------------------
--------------------------------------------------------------
--------------------------------------------------------------


TP 2 :

2-1 What are testcontainers?

Pour tester que tout marche bien comme la connection avec la bdd ou que ça build. Le test de l'intéraction avec la base de données Postgre se font dans un environnement contrôlé et éphémère.

--------------------------------------------------------------

2-2 Document your Github Actions configurations.

À chaque push ou pull request sur main et develop, ce pipeline teste le backend.

--------------------------------------------------------------

2-3 For what purpose do we need to push docker images?

On a besoin de pousser les images docker pour déployer l'application facilement dans d'autres environnement (comme des env de dev, test ou prod) sans avoir besoin de recontruire l'image chaque fois.

--------------------------------------------------------------

2-4 Document your quality gate configuration.

SonarCloud est intégré dans le job de test pour analyser la qualité du code.

Modification du job test-backend

  - name: Run SonarCloud analysis
    run: mvn -B verify sonar:sonar -Dsonar.projectKey=clefprojet -Dsonar.organization=organisationsonar -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{ secrets.SONAR_TOKEN }}
    working-directory: simple-api

Après exécution, le rapport d'analyse est disponible sur SonarCloud.

--------------------------------------------------------------
--------------------------------------------------------------
--------------------------------------------------------------


TP 3 :

3-1 Document your inventory and base commands

- Création d'un fichier "my-project/ansible/inventories/setup.yml" :

all:
  vars:
    ansible_user: admin
    ansible_ssh_private_key_file: /path/to/private/key
  children:
    prod:
      hosts: hostname_or_IP

- Test de l'inventaire avec :
ansible all -i inventories/setup.yml -m ping

- Récupération des informations de l'hôte :
ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"

- Suppression d'Apache2 avec :
ansible all -i inventories/setup.yml -m apt -a "name=apache2 state=absent" --become


--------------------------------------------------------------

3-2 Document your playbook

- hosts: all
  gather_facts: false
  become: true
  tasks:
    - name: Test de connexion
      ping:

- Exécution avec :
ansible-playbook -i inventories/setup.yml playbook.yml

Playbook pour installer Docker :
- hosts: all
  gather_facts: true
  become: true
  tasks:
    - name: Installer les paquets requis
      apt:
        name:
          - apt-transport-https
          - ca-certificates
          - curl
          - gnupg
          - lsb-release
          - python3-venv
        state: latest
        update_cache: yes
    - name: Ajouter la clé GPG de Docker
      apt_key:
        url: https://download.docker.com/linux/debian/gpg
        state: present
    - name: Ajouter le dépôt Docker
      apt_repository:
        repo: "deb [arch=amd64] https://download.docker.com/linux/debian {{ ansible_facts['distribution_release'] }} stable"
        state: present
        update_cache: yes
    - name: Installer Docker
      apt:
        name: docker-ce
        state: present
    - name: Assurer le démarrage de Docker
      service:
        name: docker
        state: started

--------------------------------------------------------------

3-3 Document your docker_container tasks configuration.

ansible-galaxy init roles/docker
- Répartition des tâches dans "roles/docker/tasks/main.yml".

Déploiement de l'application avec Docker :
- name: Démarrer l'application
  docker_container:
    name: my_app
    image: my_dockerhub_image
    state: started
    restart_policy: always
    ports:
      - "80:80"
    env:
      DATABASE_URL: "mysql://user:password@db_host/db_name"