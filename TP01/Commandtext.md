# terminal CMD

## coté network

>docker network create app-network

## coté DB

>docker run -p 8888:5000 --name database --network app-network  heavenshk/database


## coté adminer

>docker run --network=app-network -p 8080:8080 adminer


##  dockerfile de la base de données

```docker
FROM postgres:11.6-alpine

ENV POSTGRES_DB=db \
POSTGRES_USER=usr \
POSTGRES_PASSWORD=pwd

COPY /01-CreateScheme.sql /docker-entrypoint-initdb.d 

COPY /02-insertData.sql /docker-entrypoint-initdb.d
```


- Persistence

>docker run -p 8888:5000 --name database -v /tmp/data:/var/lib/postgresql/data --network app-network  heavenshk/database


## Java

```docker
FROM openjdk:11
COPY . /usr/src/myapp
WORKDIR /usr/src/myapp
RUN javac Main.java
CMD ["java", "Main"]
```
- Build :
 >docker build -t my-java-app .
- Run :
 >docker run -it --rm --name my-running-app my-java-app

- Le shell affiche : 
> "Hello World"




## Backend
```ne pas oublier de kill le processus database pour faire fonctionner sur le port 8080 ou changer de port ```

- Spring
>docker run -d --name backend -p 8080:8080 heavenshk/backend

- Docker file spring
```docker
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
>docker rm -f backend
- Api




- Apache

>docker build -t web_server .

```docker
FROM httpd:2.4
COPY index.html /usr/local/apache2/htdocs/
```

>docker run -dit --name my-running-app -p 8082:80 --network app-network web_server

Configuration en vue de faire le reverse proxy
>docker run --rm httpd:2.4 cat /usr/local/apache2/conf/httpd.conf > my-httpd.conf

-Config du dockerfile :

```docker
FROM httpd:2.4
COPY ./my-httpd.conf /usr/local/apache2/conf/httpd.conf
```



- Reverse proxy :

```docker

ServerName localhost

<VirtualHost *:80>
    ProxyPreserveHost On
    ProxyPass / http://api:8080/
    ProxyPassReverse / http://api:8080/
</VirtualHost>

```

>docker run --name my-running-app -p 80:80 --network app-network web_server


- Docker compose 

>docker-compose up
>docker-compose up --build
>docker-compose restart api

```docker

version: '3.3'
services:
  api:
    build:
      ./Javaa/simple-api-main
    networks:
      - my-network
    depends_on:
      - database

  database:
    build:
      ./postgre_bdd
    networks:
      - my-network

  httpd:
    build:
      ./http_server
    ports:
      - 80:80
    networks:
      - my-network
    depends_on:
      - api

networks:
  my-network: 


```


- Publish

> docker tag tp01_httpd heavenshk/httpd:1.0
> docker tag tp01_api heavenshk/api:1.0
>  docker tag tp01_database heavenshk/database:1.0

> docker push heavenshk/httpd:1.0
> docker push heavenshk/api:1.0
> docker push heavenshk/database:1.0



# TP2

>git add .
>git commit -m "oui"
>git push

```yml

name: CI devops 2022 CPE
on:
  #to begin you want to launch this job in main and develop
  push:
    branches: main
  pull_request:
jobs:
  test-backend:
    runs-on: ubuntu-18.04
    steps:
      #checkout your github code using actions/checkout@v2.3.3
      - uses: actions/checkout@v2.3.3

      #do the same with another action (actions/setup-java@v2) thatenable to setup jdk 11
      - name: Set up JDK 11
        uses: actions/setup-java@v2
        with:
            java-version: '11'
            distribution: 'adopt'
      #finally build your app with the latest command
      - name: Build and test with Maven
        run: mvn clean verify --file ./TP01/Javaa/simple-api-main/simple-api


```

>docker login -u heavenshk
>Secret dans github

```yml

```