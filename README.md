# Reponses aux questions

Tous les README des TP's sont situés dans le fichier Commandtext.md
#### FlorianVILLEVIEILLE
#


#### Why should we run the container with a flag -e to give the environment variables ?
> On utilise le "-e" pour permettre de passer des variables d'environnement en ligne de commande (secrets, etc). ils sont éxécutés au moment ou on passe la commande. il ne sont plus stockés directement dans un fichier texte en clair.


#### Why do we need a volume to be attached to our postgres container ?
> Pour pouvoir enregistrer nos données même après avoir détruit le contenaire.


#### 1-2 Why do we need a multistage build ? And explain each steps of this dockerfile
> Pour build le package (jdk) tout d'abord.
puis pour éxecuter le -jar avec le JRE. le JRE reste plus léger

#### Why do we need a reverse proxy ?
>Le rôle du proxy inverse va être de transmettre tous les requètes des utilisateurs au serveur mais en protégeant l'identité du serveur.
çela augmente les performances (équilibrage de charge) mais aussi la sécurité (DDOS protection)
# 

#### Why is docker-compose so important ?

#### 2-1 What are testcontainers?

> c'est une bibliothèque Java qui permet d'instancier des conteneurs dockers au sein des tests unitaires. par contre on souhaite tester une bdd postgres, on va pouvoir executer une image docker postgres
#### Why do we put our images into an online repository ?

> Pour avoir du versionning d'images
pouvoir continuer de modifier une image en ayant une toujours en production.
Egalement pour pouvoir l'utiliser dans plusieurs applications


#### mvn clean verify what is supposed to do ?
> Cela effectue des tests sur notre code. permet de vérifier la qualité du code.

#### Unit tests ? component tests ?

>Ce sont des tests permettant de tester une partie du code, ou bien une "unité". cela peut-être une fonctionnalité.

>c'est un test de composante du code, qui est fait indépendamment des autres parties, sans que cela soit intégré.
>C'est un test d'integration, interaction entre les modules (bdd) et les autres fonctionnalités


#### Secured variables, why ?
> la variable est désormais disponible à l'utilisation dans le code (secret.VARIABLE) mais elle n'est pas accessible globalement. c'est a dire que quand on essaye de grep dessus, ou de la trouver, elle n'est pas disponible.
