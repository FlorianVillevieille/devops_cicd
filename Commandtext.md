
----------
# README - TP1
Florian VILLEVIEILLE - 4IRC

Informations complémentaires :
- Dans mon architecture se trouve deux fichiers, Backend et Javaa. Le dossier Backend est l'api généré et non pas donné par vous au long du TP. Elle n'est plus utilisé par la suite, seule le dossier Javaa reste utilisé malgrè le nom "Backend" utilisé plusieurs fois
- Certains fichiers Docker/Docker-compose/Github actions ont été documentés dans un second temps, il se peut que les commentaires n'apparaissent que dans le READ-me et pas le fichier originel :)

# Base de données

## Créer un réseau :
```
docker network create app-network
```

## Base de données : Le Run
```
docker run -p 8888:5000 --name database --network app-network  heavenshk/database
```

## Adminer : Le Run

```
docker run --network=app-network -p 8080:8080 adminer
```

##  Dockerfile de la base de données

Dans ce cas, les scripts python sont lancés au début

```docker
FROM postgres:11.6-alpine

ENV POSTGRES_DB=db \
POSTGRES_USER=usr \
POSTGRES_PASSWORD=pwd

COPY /01-CreateScheme.sql /docker-entrypoint-initdb.d 

COPY /02-insertData.sql /docker-entrypoint-initdb.d
```


## Persistence des données
Nous souhaitons conserver nos données et non plus les écraser à chaque lancements.
```
docker run -p 8888:5000 --name database -v /tmp/data:/var/lib/postgresql/data --network app-network  heavenshk/database
```


# Coté Backend : Le Java

javac Main.java permet de lancer notre application en Java

## Docker file du coté Backend :
```docker
FROM openjdk:11
COPY . /usr/src/myapp
WORKDIR /usr/src/myapp
RUN javac Main.java
CMD ["java", "Main"]
```
## Build des images docker :
```
docker build -t my-java-app .
```
## Run :
```
docker run -it --rm --name my-running-app my-java-app
```
- Le shell affiche : 
> "Hello World"


```ne pas oublier de kill le processus database pour faire fonctionner sur le port 8080 ou changer de port ```

## Spring
```
docker run -d --name backend -p 8080:8080 heavenshk/backend
```
## Docker file Spring
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
Cette commande permet de supprimer les processus docker qui tournent en fond
>docker rm -f nomduprocessus /ou\ idduprocessus

# Le serveur HTTP

## Build :
```
docker build -t web_server .
```
## Docker file du serveur HTTP
```docker
FROM httpd:2.4
COPY index.html /usr/local/apache2/htdocs/
```
## Lancement de l'application Web
```
docker run -dit --name my-running-app -p 8082:80 --network app-network web_server
```

## Modification de la configuration 
```
docker run --rm httpd:2.4 cat /usr/local/apache2/conf/httpd.conf > my-httpd.conf
```
## Modification du dockerfile pour ajouter ces lignes et permettre d'utiliser le fichier de configuration initial :

```docker
FROM httpd:2.4
COPY ./my-httpd.conf /usr/local/apache2/conf/httpd.conf
```


## Docker file du serveur HTTP
Cette fois-ci, on autorise le reverse proxy. api ici correspond au docker de mon coté backend

```docker
ServerName localhost

<VirtualHost *:80>
    ProxyPreserveHost On
    ProxyPass / http://api:8080/
    ProxyPassReverse / http://api:8080/
</VirtualHost>

```
## Lancement de l'application web sur un réseau commun
```
docker run --name my-running-app -p 80:80 --network app-network web_server
```
## Quelques commandes pour manipuler docker compose 
```
>docker-compose up #lancer le docker-compose

>docker-compose up --build

>docker-compose restart api #redemarrer un des docker

>docker-compose down #eteindre les containers
```
## Fichier docker-compose
Ce fichier permet de lancer tous les processus docker en même temps, sans avoir à les démarrer un par un. Il tient compte des priorités et de chaques docker file.

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


## Publication des images que l'on utilise sur docker Hub.

Cela permet de versionner les images et qu'elles soient accessible partout. également nous pouvons la modifier pendant qu'une version qui fonctionne en est prod.
```
> docker tag tp01_httpd heavenshk/httpd:1.0

> docker tag tp01_api heavenshk/api:1.0

>  docker tag tp01_database heavenshk/database:1.0


> docker push heavenshk/httpd:1.0

> docker push heavenshk/api:1.0

> docker push heavenshk/database:1.0
```


-------------------------------------------
# READ ME - TP2

## Quelques commandes git pour gérer au mieux son dossier git.
```
>git status

>git add .

>git commit -m "message"

>git push origin master/main
```
# Github Actions
```yml
name: CI devops 2022 CPE
on:
  #to begin you want to launch this job in main and develop
  push:
    branches: main
  pull_request:
jobs:
  test-backend: #Ici on choisit l'OS
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
      #Maven est exécuté sur notre projet Java pour effectuer des tests
        run: mvn clean verify --file ./TP01/Javaa/simple-api-main/simple-api
```

Ici Maven possède un outil de review de code. selon les paramètres que l'ont choisit, il est capable d'analyser et d'en ressortir une note. généralemnt les équipes projets utilise une barre à 80% avant d'envoyer ce code en production.


>docker login -u heavenshk

Les Secrets sont stockés dans github via des variables secrètes

## Quality Gate configuration

```yml
name: CI devops 2022 CPE
on:
  #to begin you want to launch this job in main and develop
  push:
    branches: main
  pull_request:
env: #Ici on rajoute une variable d'environnement pour le github token
  GITHUB_TOKEN: ${{secrets.GITHUB_TOKEN}}
  
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
      #Des tests Sonar ont été réalisés. on peut se connecter à Sonar grâce aux credentiels dans Github.
        run: mvn -B verify sonar:sonar -Dsonar.projectKey=FlorianVillevieille_devops_cicd -Dsonar.organization=florianvillevieille -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{secrets.SONARTOKEN }} --file ./TP01/Javaa/simple-api-main/simple-api/pom.xml



 # define job to build and publish docker image
  build-and-push-docker-image:
    needs: test-backend
    # run only when code is compiling and tests are passing
    runs-on: ubuntu-latest
    # steps to perform in job
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      # login to docker avec les credentials de github secret
      - name: Login to DockerHub
        run: docker login -u ${{ secrets.USERDOCKERHUB }} -p ${{secrets.TOKENDOCKERHUB }}

      - name: Build image and push backend
        uses: docker/build-push-action@v2
        with:
          # relative path to the place where source code with Dockerfile is located
          context: ./TP01/Javaa/simple-api-main
          # Note: tags has to be all lower-case
          tags:
            ${{secrets.USERDOCKERHUB}}/tp-devops-cpe:backend
          # push action 
          # /main pour la branche sur github
          push: ${{ github.ref == 'refs/heads/main' }}

      - name: Build image and push database
        uses: docker/build-push-action@v2
        with:
          # relative path to the place where source code with Dockerfile is located
          context: ./TP01/postgre_bdd
          tags:
            ${{secrets.USERDOCKERHUB}}/tp-devops-cpe:db
          # push action
          push: ${{ github.ref == 'refs/heads/main' }}

      - name: Build image and push httpd
        uses: docker/build-push-action@v2
        with:
          # relative path to the place where source code with Dockerfile is located
          context: ./TP01/http_server
          # Note: tags has to be all lower-case
          tags:
            ${{secrets.USERDOCKERHUB}}/tp-devops-cpe:simple-frontend
          # push action
          push: ${{ github.ref == 'refs/heads/main' }}
```

Dans les "balises" context, on cherche à aller chercher les fichiers Dockerfile de chaque image

-------------------
# READ ME - TP3

## Commandes ansible :
```
ansible all -i inventory.yml -m ping
```

Il est important de noter que le -i permet de passer par un host, car les PC de CPE ne permettait pas d'ouvrir le fichier /etc/host. Il faut également une clef RSA

```yml

all:
  vars:
    ansible_user: centos
    ansible_ssh_private_key_file: /fs03/share/users/florian.villevieille/home/Workspace/id_rsa
    #Le fichier contenant la clef n'est pas stocké sur le git
  children:
    prod:
      hosts: florian.villevieille.takima.cloud

```
Cette commande permet d'obtenir des informations sur la distribution

```
ansible all -i inventory.yml -m setup -a "filter=ansible_distribution*"
```
```bash
florian.villevieille.takima.cloud | SUCCESS => {
    "ansible_facts": {
        "ansible_distribution": "CentOS", 
        "ansible_distribution_file_parsed": true, 
        "ansible_distribution_file_path": "/etc/redhat-release", 
        "ansible_distribution_file_variety": "RedHat", 
        "ansible_distribution_major_version": "8", 
        "ansible_distribution_release": "NA", 
        "ansible_distribution_version": "8", 
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    }, 
    "changed": false
}
```

## Installation d'apache
```
ansible all  -a “name=httpd state=present” --become -i inventory.yml
```
## Page HTML Via Ansible

```html
ansible all -a 'echo <html><h3>Je sais pas ce que je fais</h3></html> >> /var/www/html/index.html' --become -i inventory.yml
```
## Supprimer le serveur apache après le TD3
```

ansible all -i inventory.yml -m yum -a "name=httpd state=absent" --become
```
```bash
florian.villevieille.takima.cloud | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    }, 
    "changed": true, 
    "msg": "", 
    "rc": 0, 
    "results": [
        "Removed: mod_http2-1.15.7-3.module_el8.4.0+778+c970deab.x86_64", 
        "Removed: httpd-2.4.37-43.module_el8.5.0+1022+b541f3b1.x86_64"
    ]
}
```


## Test avec un Playbook qui éxecute un ping
```
ansible-playbook -i inventory.yml playbook.yml
```
## Playbook.yml :
```yml
- hosts: all
  gather_facts: false
  become: yes
  tasks:
    - name: Test connection
      ping:
```

# Installation de docker
```
> ansible-playbook -i inventory.yml playbook.yml
```
## Le playbook : 

Ici on spécifie les configurations d'installations pour docker.
```yml
- hosts: all
  gather_facts: false
  become: yes
# Install Docker
  tasks:
  - name: Clean packages
    command:
      cmd: dnf clean -y packages
  - name: Install device-mapper-persistent-data
    dnf:
      name: device-mapper-persistent-data
      state: latest
  - name: Install lvm2
    dnf:
      name: lvm2
      state: latest
  - name: add repo docker
    command:
      cmd: sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
  - name: Install Docker
    dnf:
      name: docker-ce
      state: present
  - name: install python3
    dnf:
      name: python3
  - name: Pip install
    pip:
      name: docker
  - name: Make sure Docker is running
    service: name=docker state=started
    tags: docker
```


Ce fichier va permettre d'appeler tous les rôles que l'ont aura au préalable configuré dans roles/"docker ou api ou front"/tasks/main.yml par exemple.

### main.yml

```yml
- hosts: all
  gather_facts: false
  become: yes

  roles:
    - docker
    - network
    - database
    - app
    - proxy
    - front
    
```


## Main.yml pour la database, un exemple
```yml
- name: DATABASE
  docker_container:
    name: database
    image: heavenshk/tp-devops-cpe:db
    state: started
    env:
      POSTGRES_DB: db
      POSTGRES_USR: usr
      POSTGRES_PASSWORD: pwd
    networks:
      - name: app-network
```

## main.yml pour l'api
```yml
- name: api
  docker_container:
    name: api
    image: heavenshk/tp-devops-cpe:backend
    networks:
      - name: app-network
```


## main.yml dans le front
```yml
- name: Create a front container
  docker_container:
    name: front
    image: heavenshk/tp-devops-cpe:simple-front
    networks: 
      - name: app-network
```
## main.yml dans le network 
```yml
- name: Create network 
  docker_network:
    name: app-network
```

## dans le httpd.conf pour autoriser le front
```yml
 
    ProxyPass / http://frontend:80/
    ProxyPassReverse / http://frontend:80/

```

