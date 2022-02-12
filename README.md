# Devops S8 2022 CPE Lyon
*Ayant fait le TP à distance à cause du covid, je me suis fais aider et conseiller par mes camarades sur place*

# TP1 

# Database

## Basics

Création d'un Dockerfile

```
FROM postgres:11.6-alpine
ENV POSTGRES_DB=db \ POSTGRES_USER=usr \ POSTGRES_PASSWORD=pwd
```

Ce fichier est placé dans un répertoire spécifique au projet

Je build et renomme mon image avec

```
docker build  . -t auremoote/postgres

docker images

docker tag 6323019f4b56 auremoote/postgres:latest
```

J'ai ensuite run

```
docker run -p 127.0.0.1:5432:5432/tcp --name postgres auremoote/postgres
```

Je n'utilise pas adminer mais l'application graphique TablePlus

Je veux créer un network pour me connecter à postgres

```
docker stop postgres auremoote/postgres

docker rm postgres auremoote/postgres

docker network create app-network

docker run -p 127.0.0.1:5432:5432/tcp --network app-network --name postgres auremoote/postgres
````

Je me connecte via l'adresse 127.0.0.1:5432 avec comme nom d'utilisateur *usr* et comme mot de passe *pwd*, à la base de données *db*

## Init database

J'ai créé des fichiers pour les scripts SQL à la racine du projet (au même endroit que mon Dockerfile) et j'ai créé le dossier *docker-entrypoint-initdb.d*.

J'ai rajouté les lignes suivantes dans le Dockerfile

```
COPY CreateScheme.sql /docker-entrypoint-initdb.d/
COPY InsertData.sql /docker-entrypoint-initdb.d/
```

## Persist Data

Pour éviter que les données soient supprimées à chaque fois que l'on supprime le container

```
docker run -p 127.0.0.1:5432:5432/tcp --network app-network -v /var/folders --name postgres auremoote/postgres
```
car je suis sur MacOS

# Backend API

## Basics

Dockerfile permettant d'éxecuter un fichier Java sous OpenJDK :
```
FROM openjdk:16-alpine3.13
ADD Main.java Main.java
RUN javac Main.java
CMD java Main
```
On build

```
docker build . -t auremoote/openjdk
```

Pour l'éxecuter on utilise donc :

```
macintosh % docker run --name apijava auremoote/openjdk
Hello World!
```
## Multistage build

Création d'un Dockerfile dans à la racine du projet simple-api

``` 
# Build 
FROM maven:3.6.3-jdk-11 AS myapp-build 
ENV MYAPP_HOME /opt/myapp 
WORKDIR $MYAPP_HOME 
COPY pom.xml . 
COPY src ./src 
RUN mvn package -DskipTests 

# Run 
FROM openjdk:11-jre 
ENV MYAPP_HOME /opt/myapp 
WORKDIR $MYAPP_HOME 
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar 

ENTRYPOINT java -jar myapp.jar
```

On run
```
macintosh % docker run --name simple-api auremoote/maven
 .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.6.3)

etc...
```

Et on peut affirmer que 
```
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar 
```
du Dockerfile permet de récupérer les fichier générés lors du build afin de réaliser le run avec l'openjdk:11

## Backend API



# HTTP Server

## Basics

Dockerfile pour web server Apache (port 80)
```
FROM httpd:2.4
```

```
docker build . -t auremoote/httpd
docker run --name http_basic -p 80:80 auremoote/httpd
```
J'ai eu une erreur que j'ai du aller résoudre dans le fichier de configuration apache2.conf
```
cd /etc/apache2/
sudo nano apache2.conf
```
```
ServerName localhost
```

*Pour retrouver les logs du serveur*
```
docker logs http_basic
```
### Configuration

Tout d'abord j'ai récupéré le fichier de config HTTPD du container via docker cp
```
docker cp http_basic:/etc/apache2/httpd.conf .
```

mod_proxy_http à ajouter dans le fichier httpd.conf

```
ServerName localhost
<VirtualHost *:80>
        ProxyPreserveHost On
        ProxyPass / http://backend-api:8080/
        ProxyPassReverse / http://backend-api:8080/
</VirtualHost>
```

```
docker run --name http_basic -p 80:80 --network app-network httpd
```

# Link application

## Docker compose

docker-compose.yaml à la racine du projet

```
version: '3.7'

services:
  apijava:
    build:
      ./APIJava/
    networks:
      - my-network
    depends_on:
      - postgres
  postgres:
    build:
      ./Postgres/
    networks:
      - my-network
    volumes:
      - db-volume:/var/folders 
  httpd:
    build:
      ./HTTPServer/ 
    ports:
      - 80:80
    networks:
      - my-network
    depends_on:
      - apijava
networks:
  my-network:
volumes:
  db-volume: {}
```

Pour le lancer
```
docker-compose up -d #start
```
```
docker-compose restart apijava
```
```
docker-compose stop
``` 
```
docker-compose rm 
```

# Publish

```
docker login
```
```
docker tag auremoote/postgres auremoote/postgres:1.0
```
```
docker push auremoote/postgres:1.0
```
# 
*Je travaille sur MacOS*
