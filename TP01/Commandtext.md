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

name: CI devops 2022 CPE
on:
  #to begin you want to launch this job in main and develop
  push:
    branches: main
  pull_request:
env:
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

      # login to docker
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


# TP3



>ansible all -i inventory.yml -m ping



>ansible all -i inventory.yml -m setup -a "filter=ansible_distribution*"

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
- Supprimer le serveur apache
>ansible all -i inventory.yml -m yum -a "name=httpd state=absent" --become

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

>ansible-playbook -i inventory.yml playbook.yml

```yml

- hosts: all
  gather_facts: false
  become: yes
  tasks:
    - name: Test connection
      ping:
```

- installation de docker

> ansible-playbook -i inventory.yml playbook.yml
- le playbook : 
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

- On ajoute ça dans le playbook pour les rôles
>   roles: - docker

- main.yml
```yml
---
# tasks file for roles/docker

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

- Main.yml pour la database, un exemple
```yml

- name: DATABASE
  docker_container:
    name: database
    image: heavenshk/database:1.0
    state: started
    env:
      POSTGRES_DB: db
      POSTGRES_USR: usr
      POSTGRES_PASSWORD: pwd
    networks:
      - name: app-network
```