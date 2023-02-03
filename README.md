# TP 1


## Questions :

- 1-1 : Dans le dockekfile de notre database, la commande `FROM` permet de définir l'image parent de l'image que l'on crée. la commande `ENV` sert à ajouter des variables. La commande `COPY` permet de copier un fichier de notre système dans notre image.

- 1-2 : Les multistage build sont utiles car permettent d'optimiser les fichiers Docker tout en les gardant faciles à lire et à maintenir.
dans le dockerfile la partie `build` va permettre de transformer nos fichiers java en fichier compilable en utilisant la jdk 17 amazon Corretto. Pour build il a besoin de copier dans l'image le pom.xml et les différentes classes présentent dans /src.
la commande `entrypoint`  est utilisée pour spécifier l'exécutable qui doit être lancé lorsqu'un conteneur est démarré à partir d'une image Docker.

- 1-3 : Il n'y a besoin que de deux commandes pour utiliser un docker compose. `docker-compose build` pour créer les images et `docker-compose up` qui va permettre de lancer les images dans des conteneurs.

- 1-4 : Dans le docker-compose on trouve plusieurs services, un pour chaque conteneur que l'on veut créer. La commande `build:` va definir le path où se trouve le dockerfile pour créer l'image. La commande `networks:` va permettre de définir des networks similaires entre des conteneurs pour qu'il puissent communiquer ensemble.

- 1-5 : La commande `docker tag my-database USERNAME/my-database:1.0` va permettre de créer une nouvelle image `USERNAME/my-database:1.0` à partir de l'image `my-database`. Après cela on utilise la commande `docker push USERNAME/my-database  
` qui permet de push l'image sur docker hub.


## Database 

On récupère les images de PostgreSQL et adminer avec un pull

- ``docker pull adminer``

Puis on crée l’image de notre database

    ``FROM postgres:14.1-alpine
    ENV POSTGRES_DB=db \
    POSTGRES_USER=usr \
    POSTGRES_PASSWORD=pwd``

On génère l'image :

- ``docker build -t database .``

On crée ensuite des containers : 

- celui de adminer : 

    ``docker run \``
    ``-p "8090:8080" \``
    ``--net=app-network \``
    ``--name=adminer \``
    ``-d \``
    ``adminer``

- puis celui de notre database :

    ``docker run -d --network app-network --name database_containeur database``

Avec `docker ps` on vérifie si les containers sont en cours d'exécution.

On peut se connecter à la BDD sur l'adresse localhost/8090

On crée les deux fichiers sql, on les ajoute dans l'image avec :

`COPY CreateScheme.sql /docker-entrypoint-initdb.d`

On supprime ensuite puis on relance image et container de database. 
On peut voir que sur localhost/8090 les tables ont bien été créés.

On peut ajouter l'option `-v /my/own/datadir:/var/lib/postgresql/data` pour que les data persistent malgré la suppression du conteneur.

## API
On crée la class main (dans le cours)
On crée un docker file avec :

`FROM openjdk:11`
`COPY Main.java Main.java`
``RUN javac Main.java``
`CMD ["java", "Main"]`

On build l'image : apijava, et on crée le container avec :
 `docker run --name api_containeur apijava`

## Multistage build

On crée une APi avec spring Initializr. 
On ajoute un dossier controller et un fichier controller.

on build l'image avec :
 
 ```# Build
FROM maven:3.8.6-amazoncorretto-17 AS myapp-build
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY pom.xml .
COPY src ./src
RUN mvn package -DskipTests

# Run
FROM amazoncorretto:17
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

ENTRYPOINT java -jar myapp.jar
```
On build l'image et on crée le container avec la commande :

`docker run -p "8081:8080" --name multiapi_containeur multiapi`

Si on se connecte à localhost/8081 on voit que notre Api marche

## Api avec BDD

On récupère le zip du projet sur le tp.
On modifie le fichier yml pour ajouter les identifiants de la bdd. 
On crée le même dockerfile que précédemment.
On build l'image puis on crée le container avec la commande :

`docker run -p "8081:8080 --network app-network --name api_bdd_containeur apibdd"`

On peut avoir accès à la bdd par l'api

## HTTP serveur, reverse-proxy

Nous avons récupéré la correction du cours et avons réussi à l'adapter et à l'éxecuter avec docker compose.

## docker-compose

Un docker compose est un fichier de configuration qui va nous permettre de générer toutes les images et conteneurs directement avec les commandes `docker-compose build` & `docker-compose up`


# TP2

## Questions : 

- 2-1 : Il s'agit simplement de bibliothèques Java qui vous permettent d'exécuter un ensemble de conteneurs Docker pendant les tests.

- 2-2 : Dans le fichier de configuration on retrouve la branche sur laquelle l'action doit avoir lieu, ici `main` puis les différents jobs à faire. 
Chaque jobs à differents `steps`. On retrouve des :
    - `uses` définit des actions à faire.
    - `run` permet de lancer des commandes sur le terminal.
    - `with` permet d'ajouter des compléments aux uses. 


## Github

Tout d'abord on crée le dossier .github/workflows et on ajoute un fichier main.yml.

A l'interieur on définit la branche sur laquelle on va effectuer des taches et ensuite on ajoute les différents ``jobs`` à réaliser.

Le job `test-backend` va nous permettre de vérifier que le backend fonctionne en lançant un build.

## Docker Hub

On va ensuite créer un job permettant de build nos images et de les ajouter à Docker Hub.
Pour se faire on doit ajouter nos identifiants de Docker Hub dans les `actions secrets` de Github, et ensuite les utiliser comme variables.

## SonarCloud

Dans un premier temps on crée une organisation sur sonar et on récupère les clés de projet pour les ajouter sur Github. 
On doit ensuite modifier la commande `mvn clean verifiy`pour qu'un rapport Sonar soit créé lorsque le backend est build.

`mvn -B verify sonar:sonar -Dsonar.projectKey=devops-2023 -Dsonar.organization=devops-school -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{ secrets.SONAR_TOKEN }}  --file ./simple-api/pom.xml
`

Lorsqu'on va sur sonar on peut voir les `quality gate`.


# TP 3

## Questions : 

- 3-1 : Dans `setup.yml`, le groupe `all` contient des variables pour le nom d'utilisateur (ansible_user : centos) et le chemin vers la clé SSH privée (ansible_ssh_private_key_file : /path/to/private/key) qui seront utilisés pour se connecter aux hôtes. le groupe `prod` spécifie un hôte avec son adresse ip. la commande `ansible all -i inventories/setup.yml -m ping` permet de tester à l'aide d'un ping si les hôtes définient dans setup.yml sont accessibles. La commmande `ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"` permet d'avoir les informations sur la distribution des hôtes. La commmande `ansible all -i inventories/setup.yml -m yum -a "name=httpd state=absent"` permet de désinstaller le paquet `httpd` des hôtes.

- 3-2 : Le playbook permet l'installation de docker sur CentOS. Dans un premier temps il nettoie les paquets avec la commande `dnf clean`. il installe ensuite les paquets préréquis `device-mapper-persistent-data` et `lvm2`. Il ajoute un repository Docker avec la commande `sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo`. Il installe Docker et python3 avec `dnf` et les paquets docker avec `pip`. Enfin, il vérifie que le service Docker est en cours d'exécution avec `service`.

## Ansible

On ajoute le fichier setup.yml dans le dossier ansible. 
On le configure, on lui ajoute le chemin complet de la clé RSA et notre host qui est le serveur envoyé par mail. 
Puis la commande `ansible all -i inventories/setup.yml -m ping` fonctionne.
On crée ensuite des playbook, les playbooks permettent de faire des actions, par exemple d'installer docker.
on crée ensuite des rôles et on ajoute dans chaque roles des tasks spécifiques, pour chaque partie de notre projet.
Le playbook appellera ensuite les différents rôles.
Si on a pas fait d'erreurs on voit que le serveur est bien lancé.






