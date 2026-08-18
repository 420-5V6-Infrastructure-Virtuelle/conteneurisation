---
title: DockerFiles
weight: 2070
---

## Qu'est-ce qu'un Dockerfile ?
Un Dockerfile est simplement un fichier texte qui enchaîne des instructions destinées à installer tout ce dont notre application a besoin pour fonctionner.

La liste d’instructions disponibles est assez courte, voire une dizaine d'instructions, mais c’est suffisant pour construire une image complète.

On peut (je dirais, "On doit") y ajouter des commentaires pour documenter ce que l’on fait afin aussi de facilité.

</br>

## Créer une image en utilisant un Dockerfile

- Jusqu'ici nous avons utilisé des images toutes prêtes téléchargées sur Docker Hub

- Une des fonctionnalités principales de Docker est de pouvoir facilement construire des images à partir d'un simple fichier texte : **le Dockerfile**.

</br>

## Le processus de build Docker avec un Dockerfile

- Une image Docker ressemble un peu à un template de VM, car on peut penser à un Linux figé dans un état.<!-- - En réalité, c'est assez différent : il s'agit uniquement d'un système de fichier (par couches ou _layers_) et d'un manifeste JSON (des métadonnées). -->
- On construit les images à partir d'un fichier `Dockerfile` en décrivant procéduralement (étape par étape) la construction.

- Les images sont créées en empilant de nouvelles couches sur une image existante grâce à un système de fichiers qui fait du _union mount_.

  - On empile les couches, chacune résultant d’une instruction, à partir d’une image de base.
![](../../../images/docker/docker-image-layers.CYawgeO-_4okP.svg)

- Chaque nouveau build génère une nouvelle image dans le répertoire des images (`/var/lib/docker/images`) (attention ça peut vite prendre énormément de place)

- Privilégiez les images slim ou alpine pour réduire la taille finale. (Contiens une version très légère de Linux)

![](../../../images/ops-images-dockerfile.svg)

### Exemple de Dockerfile :

```Dockerfile
FROM python:3.11-slim
COPY . /app
RUN pip install -r /app/requirements.txt
```
Voici ce que fait Docker :

  - 1re couche : image de base python:3.11-slim
  - 2e couche : copie de vos fichiers dans /app
  - 3e couche : installation des dépendances Python

  Chaque couche est immutabilité et cacheable. Si on relance le build sans changer les fichiers copiés, Docker réutilisera les couches précédentes.

### Le cache, votre allié pour des builds rapides

L'ordre des instructions dans le Dockerfile impacte directement le cache. Placez les instructions qui changent rarement (installation de dépendances système) en haut, et celles qui changent souvent (copie du code source) en bas. Ainsi, le cache est réutilisé au maximum.


### Comment construire l'image

- La commande pour construire l'image est à partir d'un Dockerfile :

```
docker build [-t tag] [-f dockerfile] <build_context>
```

- généralement pour construire une image on se place directement dans le dossier avec le `Dockerfile` et les éléments de contexte nécessaire (programme, config, etc), le contexte est donc le caractère **`.`**, il est obligatoire de préciser un contexte.

- exemple : `docker build -t mon-debian .`


Voici un exemple complet de l'installation d'un projet Python

```Dockerfile
# Image de base : on utilise une distribution Alpine légère
FROM alpine:3.5

# Définition du répertoire de travail pour centraliser l'application
# Répertoire dans lequel toutes les commandes suivantes s'exécuteront
WORKDIR /usr/src/app

# Installation de Python 2 et de pip, puis mise à jour de pip (apk fait la même chose que apt en plus léger ... apk est le gestionnaire de paquets d’Alpine Linux )
# (Regrouper les installations permet de limiter le nombre de couches de l'image)
RUN apk add --update py2-pip && \
    pip install --upgrade pip

# Installation des dépendances Python
# On copie d'abord uniquement le fichier requirements.txt pour mettre en cache 
# C'est le fichier des dépendances Python
COPY requirements.txt ./

# Installation des dépendances Python sans cache pour réduire la taille de l'image
RUN pip install --no-cache-dir -r requirements.txt

# Copie des fichiers de l'application
# Ce sont les sources et les templates nécessaires à l'application python qui a été développé
COPY app.py ./
COPY templates/index.html ./templates/

# Indication du port sur lequel l'application écoute à l'intérieur du conteneur
EXPOSE 5000

# Commande de démarrage pour lancer l'application Python
CMD ["python", "app.py"]
```

---

## Les Instructions 

</br>

### `FROM`

- L'image de base à partir de laquelle est construite l'image actuelle.
```Dockerfile
FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]
```
</br>

### `RUN`

- Permets de lancer une commande shell (installation, configuration).
```Dockerfile
RUN apk update
RUN apk add nginx
```

</br>

###  `ADD` ou `COPY`

- Permets d'ajouter des fichiers depuis le contexte de build à l'intérieur du conteneur.
- Généralement utilisé pour ajouter le code du logiciel en cours de développement et sa configuration au conteneur.
- Ces deux instructions ont des petites différences subtiles : les options de `COPY` sont plus complètent, et `ADD` permettent d'extraire automatiquement des archives locales (.tar).

```Dockerfile
COPY [--chown=<user>:<group>] [--chmod=<perms>] <src>... <dest>
#ou
COPY [--chown=<user>:<group>] [--chmod=<perms>] ["<src>",... "<dest>"]

ADD [--chown=<user>:<group>] [--chmod=<perms>] [--checksum=<checksum>] <src>... <dest>
# ou
ADD [--chown=<user>:<group>] [--chmod=<perms>] ["<src>",... "<dest>"]
```

</br>

###  `CMD`

- Généralement à la fin du `Dockerfile` : elle permet de préciser la commande par défaut lancé à la création d'une instance du conteneur avec `docker run`. on l'utilise avec une liste de paramètres

L'instruction `CMD` a trois formes :
* `CMD ["executable","param1","param2"]` (*exec form*, forme à préférer)
* `CMD ["param1","param2"]` (combinée à une instruction `ENTRYPOINT`)
* `CMD command param1 param2` (*shell form*)

```Dockerfile
CMD ["echo 'Conteneur démarré'"]
```
</br>

### `CMD` et `ENTRYPOINT`

- Précise le programme de base avec lequel sera lancée la commande
- La principale différence entre CMD et ENTRYPOINT est que les commandes fournies par CMD peuvent être remplacées, alors que celles fournies par ENTRYPOINT ne le peuvent pas

```Dockerfile
CMD ["executable","param1","param2"]

ENTRYPOINT ["executable", "param1", "param2"]

```


* Ne surtout pas confondre avec `RUN` qui exécute une commande Dockerfile uniquement pendant la construction de l'image.
* La différence entre `CMD` et `ENTRYPOINT` c'est que cmd peut avoir des paramètres dynamiques et entrepoint doit avoir des paramètres statiques

Exemple de combinaison de `CMD` et `ENTRYPOINT`

```Dockerfile
FROM alpine:3.20
ENTRYPOINT ["echo"]
CMD ["Conteneur démarré"]
```
```bash
docker build -t demo .
docker run demo # résultat : Bonjour Docker!
docker run demo "Yo patate !" # résultat : Yo patate !
```

</br>

###  `ARG` 
- L'instruction `ARG`  est la seule instruction qui peut précéder l'instruction FROM. 
- Elle permet de définir des variables qui peuvent être transmises au moment de la construction de l'image.
```Dockerfile
ARG <name>[=<default value>]
#exemple
ARG version=1.15.3-alpine@sha256:829a63ad2b1389e393e5decf5df25860347d09643c335d1dc3d91d25326d3067

RUN echo ${version}
```

</br>

### Les variables
On peut utiliser des variables d'environnement dans les Dockerfiles. La syntaxe est `${...}`.
Exemple :
```Dockerfile
FROM busybox
ENV FOO=/bar
WORKDIR ${FOO}    # WORKDIR /bar
ADD . $FOO        # ADD . /bar
COPY \$FOO /quux  # COPY $FOO /quux
```

---

###  `ENV`

- Une façon recommandée de configurer vos applications Docker est d'utiliser les variables d'environnement UNIX, ce qui permet une configuration "au _runtime_".
```Dockerfile
ENV <key>=<value> ...
#exemple
ENV PGDATA=/data
```

Se référer au [mode d'emploi](https://docs.docker.com/engine/reference/builder/#environment-replacement) pour la logique plus précise de fonctionnement des variables.

</br>

### `USER`

- L'instruction USER définit l'utilisateur ou l'UID et éventuellement le groupe d'utilisateurs ou le GID à utiliser pour le reste de l'étape en cours. 
- L'utilisateur spécifié est utilisé pour les instructions RUN et, au moment de l'exécution de l'image de conteneur.

```Dockerfile
USER <user>[:<group>]
#ou
USER UID[:GID]
```

- Un exemple permettant de spécifier l'UID et le GID via des ARG's :

```Dockerfile
# Utilise une image Alpine précise via son digest pour garantir une version immuable
FROM alpine@sha256:d7342993700f8cd7aba8496c2d0e57be0666e80b4c441925fc6f9361fa81d10e

# Définit l’UID du futur utilisateur (par défaut 1000)
ARG USER_UID=1000

# Définit le GID, identique à l’UID sauf si modifié
ARG USER_GID=${USER_UID}

# Crée un groupe et un utilisateur, installe sudo, et configure l’accès root sans mot de passe
RUN groupadd --gid $USER_GID $USERNAME \        # Crée un groupe avec le GID fourni
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME \   # Crée l’utilisateur avec home directory
    && apk update \                                             # Met à jour l’index des paquets (APT — attention, Alpine n’utilise pas APT)
    && apk install sudo \                                    # Installe sudo
    && echo "$USERNAME ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/$USERNAME \  # Autorise sudo sans mot de passe
    && chmod 0440 /etc/sudoers.d/$USERNAME                      # Sécurise le fichier sudoers

# Définit l’utilisateur par défaut pour les prochaines instructions
USER $USERNAME
```
</br>

### `WORKDIR`

L'instruction `WORKDIR` définit le répertoire de travail pour toutes les instructions qui la suivent dans le Dockerfile. Si le répertoire n'existe pas, il sera créé.

```Dockerfile
WORKDIR /path/to/workdir
```
</br>

### `HEALTHCHECK`

`HEALTHCHECK` permet de vérifier si l'app contenue dans un conteneur est en bonne santé.

```Dockerfile
HEALTHCHECK CMD curl --fail http://localhost:5000/health
```

</br>

###  `VOLUME` 

- L'instruction [VOLUME](../4_volumes/#les-volumes-docker-via-la-sous-commande-volume) crée un point de montage 

```Dockerfile
VOLUME ["/data"]
```

</br>

### `EXPOSE` 

- L'instruction [EXPOSE] informe le moteur de conteneur que le conteneur écoute sur les ports réseau spécifiés au moment de l'exécution. 
- Vous pouvez spécifier le protocole TCP ou UDP, TCP étant la valeur par défaut.

```Dockerfile
EXPOSE <port> [<port>/<protocol>...]
```
</br>

### `BUILD` - Lancer la construction

- La commande pour lancer la construction d'une image est :

```Dockerfile
docker build [-t <tag:version>] [-f <chemin_du_dockerfile>] <contexte_de_construction>
```

- Lors de la construction, Docker télécharge l'image de base. On constate plusieurs téléchargements en parallèle.

- Il lance ensuite la séquence des instructions du Dockerfile.

- Observez l'historique de construction de l'image avec `docker image history <image>`

- Il lance ensuite la série d'instructions du Dockerfile et indique un *hash* pour chaque étape.
  - C'est le *hash* correspondant à un *layer* de l'image

---

## Documentation
- Il existe de nombreuses autres instructions possibles très clairement décrites dans la documentation officielle : [https://docs.docker.com/engine/reference/builder/](https://docs.docker.com/engine/reference/builder/)
---


## Optimiser la création d'images

- Les images Docker ont souvent une taille de plusieurs centaines de **mégaoctets** voire parfois **gigaoctets**. `docker image ls` permet de voir la taille des images.
- Or, on construit souvent plusieurs dizaines de versions d'une application par jour (souvent automatiquement sur les serveurs d'intégration continue).

  - L'espace disque devient alors un sérieux problème.

- Le principe de Docker est justement d'avoir des images légères, car on va créer beaucoup de conteneurs (un par instance d'application/service).

- De plus on télécharge souvent les images depuis un registry, ce qui consomme de la bande passante.

> La principale **bonne pratique** dans la construction d'images est de **limiter leur taille au maximum**.

---

## Limiter la taille d'une image

- Choisir une image Linux de base **minimale**:

  - Une image `ubuntu` complète pèse déjà presque une soixantaine de mégaoctets.
  - mais une image trop rudimentaire (`busybox`) est difficile à déboguer et peu bloquer pour certaines tâches à cause de binaires ou de bibliothèques logicielles qui manquent (compilation par exemple).
  - Souvent on utilise des images de base construites à partir de `alpine` qui est un bon compromis (6 mégaoctets seulement et un gestionnaire de paquets `apk`).
  - Par exemple `python3` est fourni en version `python:alpine` (99 Mo), `python:3-slim` (179 Mo) et `python:latest` (918 Mo).
---

## Créer des conteneurs personnalisés

- Il n'est pas nécessaire de partir d'une image Linux vierge pour construire un conteneur.

- On peut utiliser la directive `FROM` avec n'importe quelle image.

- De nombreuses applications peuvent être configurées en étendant une image officielle
- _Exemple : une image Wordpress déjà adaptée à des besoins spécifiques._

- L'intérêt ensuite est que l'image est disponible préconfigurée pour construire ou mettre à jour une infrastructure, ou lancer plusieurs instances (plusieurs conteneurs) à partir de cette image.

- C'est grâce à cette fonctionnalité que Docker peut être considéré comme un outil d'_infrastructure as code_.

- On peut également prendre une sorte de snapshot du conteneur (de son système de fichiers, pas des processus en train de tourner) sous forme d'image avec `docker commit <container> <image>` et `docker push`.

---
## Autre exemple complet et bien commenté d'un Docker file

```Dockerfile
###############################################
# 1. Image de base
###############################################

# Alpine : image légère, rapide, idéale pour comprendre Docker
FROM alpine:3.20



###############################################
# 2. Variables de construction (build arguments)
###############################################

# UID du futur utilisateur non-root.
# 1000 correspond généralement au premier utilisateur “humain” sur Linux.
# Cela évite que le conteneur crée des fichiers appartenant à root sur l’hôte.
# Bonne pratique : exécuter les applications Docker avec un utilisateur non-root.
ARG USER_UID=1000

# GID du groupe principal de l’utilisateur.
# UID et GID sont deux identités distinctes dans Linux (utilisateur vs groupe).
# On les sépare pour garder un Dockerfile flexible et compatible avec différents systèmes.
# Même si la valeur est identique, cela évite des problèmes de permissions avec des volumes.
ARG USER_GID=${USER_UID}

# Nom de l’utilisateur
ARG USERNAME=appuser



###############################################
# 3. Installation des dépendances
###############################################

# Mise à jour des dépôts + installation de bash et curl
RUN apk update \
    && apk add --no-cache bash curl



###############################################
# 4. Création d’un utilisateur non-root
###############################################

# Création du groupe et de l’utilisateur avec UID/GID définis plus haut
RUN addgroup -g $USER_GID $USERNAME \
    && adduser -D -u $USER_UID -G $USERNAME $USERNAME



###############################################
# 5. Copie des fichiers dans l’image
###############################################

# On copie un script de démarrage dans l’image
COPY ./start.sh /usr/local/bin/start.sh



###############################################
# 6. Permissions
###############################################

# On rend le script exécutable
RUN chmod +x /usr/local/bin/start.sh



###############################################
# 7. Définition de l’utilisateur par défaut
###############################################

# Toutes les commandes suivantes seront exécutées par l’utilisateur non-root
USER $USERNAME



###############################################
# 8. Définition du répertoire de travail
###############################################

# Répertoire de travail par défaut
WORKDIR /home/$USERNAME



###############################################
# 9. Commande de démarrage du conteneur
###############################################

# Le conteneur exécutera ce script au démarrage
CMD ["/usr/local/bin/start.sh"]
```

## Valider son dockerfile

https://blog.stephane-robert.info/docs/conteneurs/outils/hadolint/

https://blog.stephane-robert.info/docs/securiser/outils/dockle/

## Bonne pratique

https://blog.stephane-robert.info/docs/conteneurs/images-conteneurs/dockerfile-bonnes-pratiques/

https://blog.stephane-robert.info/docs/conteneurs/images-conteneurs/optimiser-taille-image/


