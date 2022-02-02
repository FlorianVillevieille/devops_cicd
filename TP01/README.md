# Compte rendu TP1
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

# TP2

#### Why is docker-compose so important ?

#### 2-1 What are testcontainers?

#### Why do we put our images into an online repository ?

> pour avoir du versionning d'images

 



#### Secured variables, why ?

#### For what purpose do we need to push docker images?