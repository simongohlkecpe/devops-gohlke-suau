# TP 1


## question :

- 1-1 : Dans le dockekfile de notre database, la commande `FROM` permet de definir l'image parent de l'image que l'on crée. la commande `ENV` sert a ajouter des variables. La commande `COPY` permet de copier un fichier de notre système dans notre image.

- 1-2 : Les multistage build sont utiles car permettent d'optimiser les fichiers Docker tout en les gardant faciles à lire et à maintenir.
dans le dockerfile la partie `build` va permettre de transformer nos fichier java en fichier compilable en utilisant la jdk 17 amazon Corretto. Pour build il a bsoin de copier dans l'image le pom.xml et les diffentes classes presentent dans /src.
la commande `entrypoint`  est utilisée pour spécifier l'exécutable qui doit être lancé lorsqu'un conteneur est démarré à partir d'une image Docker.

- 1-3 : Il n'y a besoin que de deux commandes pourutiliser un docker compose. `docker-compose build` pour créer les images et `docker-compose up` qui va permettre de lancer les images dans des conteneurs.

- 1-4 : dans le docker-compose on trouve plusieurs services, un pour chaque containeur que l'on veut créer. La commande `build:` va definir le path ou se trouve le dockerfile pour créer l'image. La commande `networks:` va permmetrre de definir des networks similaire entre des conteneurs pour qu'il puissent communiquer ensemble.

- 1-5 :


## Database 

on recupere les images de PostgreSQL et adminer avec un pull

- ``docker pull adminer``

Puis on crée l’image de notre database

    ``FROM postgres:14.1-alpine
    ENV POSTGRES_DB=db \
    POSTGRES_USER=usr \
    POSTGRES_PASSWORD=pwd``

on genère l'image :

- ``docker build -t database .``

On crée ensite des containers : 

- celui de adminer : 

    ``docker run \``
    ``-p "8090:8080" \``
    ``--net=app-network \``
    ``--name=adminer \``
    ``-d \``
    ``adminer``

- puis celle de notre database :

    ``docker run -d --network app-network --name database_containeur database``

avec `docker ps`

On peut se connecter à la BDD sur l'adresse localhost/8090

on crée les deux fichiers sql, on les ajoutes dans l'image avec :

`COPY CreateScheme.sql /docker-entrypoint-initdb.d`

On supprime ensuite puis on relance image et containeur de database. 
On peut voir que sur localhost/8090 les tables ont bien étés créés

On peut ajouter l'option `-v /my/own/datadir:/var/lib/postgresql/data` pour que les data persiste malgré la suppresion du containeur.

## API
on crée la class main (dans le cours)
on crée un docker file avec :

`FROM openjdk:11`
`COPY Main.java Main.java`
``RUN javac Main.java``
`CMD ["java", "Main"]`

on build l'image : apijava, et on crée le containeur avec :
 `docker run --name api_containeur apijava`

## Multistage build

on crée un APi avec spring Initializr
on ajoute un dossier controller et un fichier controller

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
on build l'image et on crée le containeur avec la commande :

`docker run -p "8081:8080" --name multiapi_containeur multiapi`

Si on se connect à localhost/8081 on voit que notre Api marche

## Api avec BDD

On recupère le zip du projet sur le tp
on modifie le fichier yml pour ajouter les identifiants de la bdd 
on crée le même dockerfile que précédemment
on build l'image puis on crée le containeur avec la commande :

`docker run -p "8081:8080 --network app-network --name api_bdd_containeur apibdd"`

on peut avoir accès a la bdd par l'api

## HTTP serveur, reverse-proxy

Nous avons recupéré la correction du cours et avons reussi à l'adapter et à le faire composer

## docker-compose

Un docker compose est un fichier de configuration qui va nous permettre de générer toutes les images et conteneurs directement avec les commandes `docker-compose build` & `docker-compose up`


# TP2

## question : 

- 2-1 : Il s'agit simplement de bibliothèques Java qui vous permettent d'exécuter un ensemble de conteneurs Docker pendant les tests.

- 2-2 : Dans le fichier de configuration on retrouve la branche sur laquel l'action doit avoir lieu, ici `main` puis les differents jobs à faire. 
Chaque jobs à differentes `steps` . On retrouve des :
    - `uses` defini des actions à faire.
    - `run` permet de lancer des commandes sur le terminal.
    - `with` permet d'ajouter des complements au uses. 


## Github

tout d'abord on crée le dossier .github/workflows et on ajoute un fichier main.yml

a l'interieur on defini la branhe sur laquelle on va effectuer des taaches et ensuite on ajoute les differents ``jobs`` à réaliser

le job `test-backend` va nous permettre de verifier que le backend fonctionne en lancant un build

## Docker Hub

On va ensuite crée un job permettant de build nos images et de les ajouter à Docker Hub.
Pour se faire on doit ajouter nos identifiants de DOcker Hub dans les `actions secrets` de Github, et ensuite les utiliser comme variables.

## SonarCloud

Dans un premier temps on crée une organisation sur sonar et on recupere les clés de projet pour les ajouter sur Github. 
On doit ensuite modifier la commande `mvn clean verifiy`pour qu'un rapport Sonar soit crée lorsque le bzckend est build.

`mvn -B verify sonar:sonar -Dsonar.projectKey=devops-2023 -Dsonar.organization=devops-school -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{ secrets.SONAR_TOKEN }}  --file ./simple-api/pom.xml
`

lorsqu'on va sur sonar on peut voir les `quality gate`.


# TP 3

## question : 

- 3-1 : 

- 3-2 :

## Ansible

on ajoute le fichier setup.yml dans le dossier ansible 
On le configure, on lui ajoute le chemin complet de la clé RSA et notre host qui est le serveur envoyé par mail. 
Puis la commande `ansible all -i inventories/setup.yml -m ping` fonctionne.
On crée ensuite des playbook, les playbooks permettent de faire des actions, par exemple d'installer docker.
on crée ensuite des roles et on ajoute dans chaque roles des tasks spécifiques, pour chaque partie de notre projet.
Le playbook appellera ensuite les different roles.
SI on a pas fait d'erreur on voit que le serveur est bien lancé.






