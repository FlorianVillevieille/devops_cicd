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