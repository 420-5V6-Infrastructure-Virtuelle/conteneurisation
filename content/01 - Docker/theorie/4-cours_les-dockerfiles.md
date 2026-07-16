---
title: DockerFiles
weight: 2070
---

## Qu'est-ce qu'un Dockerfile ?
Un Dockerfile est simplement un fichier texte qui enchaîne des instructions destinées à installer tout ce dont notre application a besoin pour fonctionner.

La liste d’instructions disponibles est assez courte, voire une dizaine d'instructions, mais c’est suffisant pour construire une image complète.

On peut (je dirais, "On doit") y ajouter des commentaires pour documenter ce que l’on fait.

## Créer une image en utilisant un Dockerfile

- Jusqu'ici nous avons utilisé des images toutes prêtes.

- Une des fonctionnalités principales de Docker est de pouvoir facilement construire des images à partir d'un simple fichier texte : **le Dockerfile**.

## Le processus de build Docker avec un Dockerfile

- Une image Docker ressemble un peu à un template de VM, car on peut penser à un Linux figé dans un état.<!-- - En réalité c'est assez différent : il s'agit uniquement d'un système de fichier (par couches ou _layers_) et d'un manifeste JSON (des métadonnées). -->
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

---

Voici un exemple complet de l'installation d'un projet python

```Dockerfile
# Image de base : on utilise une distribution Alpine légère
FROM alpine:3.5

# Définition du répertoire de travail pour centraliser l'application
WORKDIR /usr/src/app

# Installation de Python 2 et de pip, puis mise à jour de pip
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

### Instruction `FROM`

- L'image de base à partir de laquelle est construite l'image actuelle.

### Instruction `RUN`

- Permets de lancer une commande shell (installation, configuration).

### Instruction `ADD` ou `COPY`

- Permets d'ajouter des fichiers depuis le contexte de build à l'intérieur du conteneur.
- Généralement utilisé pour ajouter le code du logiciel en cours de développement et sa configuration au conteneur.
- Ces deux instructions ont des petites différences subtiles : les options de `COPY` sont plus complètent, et `ADD` permettent de télécharger et dézipper un fichier disponible à une URL distante.
---

### Instruction `CMD`

- Généralement à la fin du `Dockerfile` : elle permet de préciser la commande par défaut lancé à la création d'une instance du conteneur avec `docker run`. on l'utilise avec une liste de paramètres

```Dockerfile
CMD ["echo 'Conteneur démarré'"]
```

### Instruction `ENTRYPOINT`

- Précise le programme de base avec lequel sera lancée la commande

```Dockerfile
ENTRYPOINT ["/usr/bin/python3"]
```

### `CMD` et `ENTRYPOINT`

* Ne surtout pas confondre avec `RUN` qui exécute une commande Bash uniquement pendant la construction de l'image.

L'instruction `CMD` a trois formes :
* `CMD ["executable","param1","param2"]` (*exec form*, forme à préférer)
* `CMD ["param1","param2"]` (combinée à une instruction `ENTRYPOINT`)
* `CMD command param1 param2` (*shell form*)


Si l'on souhaite que notre container lance le même exécutable à chaque fois, alors on peut opter pour l'usage d'`ENTRYPOINT` en combination avec `CMD`.

---

### Instruction `ENV`

- Une façon recommandée de configurer vos applications Docker est d'utiliser les variables d'environnement UNIX, ce qui permet une configuration "au _runtime_".

---

### Instruction `HEALTHCHECK`

`HEALTHCHECK` permet de vérifier si l'app contenue dans un conteneur est en bonne santé.

```bash
HEALTHCHECK CMD curl --fail http://localhost:5000/health
```

---

### Les variables
On peut utiliser des variables d'environnement dans les Dockerfiles. La syntaxe est `${...}`.
Exemple :
```Dockerfile
FROM busybox
ENV FOO=/bar
WORKDIR ${FOO}   # WORKDIR /bar
ADD . $FOO       # ADD . /bar
COPY \$FOO /quux # COPY $FOO /quux
```

Se référer au [mode d'emploi](https://docs.docker.com/engine/reference/builder/#environment-replacement) pour la logique plus précise de fonctionnement des variables.
## Documentation

- Il existe de nombreuses autres instructions possibles très clairement décrites dans la documentation officielle : [https://docs.docker.com/engine/reference/builder/](https://docs.docker.com/engine/reference/builder/)

---

## Lancer la construction

- La commande pour lancer la construction d'une image est :

```bash
docker build [-t <tag:version>] [-f <chemin_du_dockerfile>] <contexte_de_construction>
```

- Lors de la construction, Docker télécharge l'image de base. On constate plusieurs téléchargements en parallèle.

- Il lance ensuite la séquence des instructions du Dockerfile.

- Observez l'historique de construction de l'image avec `docker image history <image>`

- Il lance ensuite la série d'instructions du Dockerfile et indique un *hash* pour chaque étape.
  - C'est le *hash* correspondant à un *layer* de l'image

---

<!-- 
# Les layers et la mise en cache

- **Docker construit les images comme une série de "couches" de fichiers successives.**

- On parle d'**Union Filesystem,** car chaque couche (de fichiers) écrase la précédente.

![](../../../images/overlay_constructs.jpg)
![](../../../images/OverlayFS_Image.png) 


- Chaque couche correspond à une instruction du Dockerfile.

- `docker image history <conteneur>` permet d'afficher les layers, leur date de construction et taille respectives.

- Ce principe est au cœur de l'**immutabilité** des images Docker.

- Au lancement d'un container, le Docker Engine rajoute une nouvelle couche de filesystem "normal" read/write par dessus la pile des couches de l'image.

- `docker diff <container>` permet d'observer les changements apportés au conteneur depuis le lancement.


---
-->



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

<!-- - Limiter le nombre de commandes de modification du conteneur :
  -  -->

<!-- 
## Les multistages builds

Quand on tente de réduire la taille d'une image, on a recours à un tas de techniques. Avant, on utilisait deux `Dockerfile` différents : un pour la version prod, légère, et un pour la version dev, avec des outils en plus. Ce n'était pas idéal.
Par ailleurs, il existe une limite du nombre de couches maximum par image (42 layers). Souvent on enchaînait les commandes en une seule pour économiser des couches (souvent, les commandes `RUN` et `ADD`), en y perdant en lisibilité.

Maintenant on peut utiliser les multistage builds.

Avec les multistages builds, on peut utiliser plusieurs instructions `FROM` dans un Dockerfile. Chaque instruction `FROM` utilise une base différente.
On sélectionne ensuite les fichiers intéressants (des fichiers compilés par exemple) en les copiant d'un stage à un autre.

Exemple de `Dockerfile` utilisant un multistage build :

```Dockerfile
FROM golang:1.7.3 AS builder
WORKDIR /go/src/github.com/alexellis/href-counter/
RUN go get -d -v golang.org/x/net/html
COPY app.go .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /go/src/github.com/alexellis/href-counter/app .
CMD ["./app"]
```
-->
---

## Créer des conteneurs personnalisés

- Il n'est pas nécessaire de partir d'une image Linux vierge pour construire un conteneur.

- On peut utiliser la directive `FROM` avec n'importe quelle image.

- De nombreuses applications peuvent être configurées en étendant une image officielle
- _Exemple : une image Wordpress déjà adaptée à des besoins spécifiques._

- L'intérêt ensuite est que l'image est disponible préconfigurée pour construire ou mettre à jour une infrastructure, ou lancer plusieurs instances (plusieurs containers) à partir de cette image.

- C'est grâce à cette fonctionnalité que Docker peut être considéré comme un outil d'_infrastructure as code_.

- On peut également prendre une sorte de snapshot du conteneur (de son système de fichiers, pas des processus en train de tourner) sous forme d'image avec `docker commit <image>` et `docker push`.

---
<!-- 
# Publier des images vers un registry privé

- Généralement les images spécifiques produites par une entreprise n'ont pas vocation à finir dans un dépôt public.

- On peut installer des **registries privés**.

- On utilise alors `docker login <adresse_repo>` pour se logger au registry et le nom du registry dans les `tags` de l'image.

- Exemples de registries :
  - **Gitlab** fournit un registry très intéressant, car intégré dans leur workflow DevOps.




## Le design pattern de l'architecture "microservice" (multiconteneurs) : [12factor.net](https://12factor.net)

[12factor.net](https://12factor.net)

- faire en sorte que le conteneur soit agnostique de l'environnement :
  - s'assure qu'il a les services dont il a besoin
  - config dans des volumes ou des variables d'environnement
  - le plus "stateless" possible
  - interagit avec d'autres services via des ports réseau
  - log bien via son process principal
  - se lance vite et s'éteint proprement rapidement
  - basé sur la même image pour la prod et le dev (ou au mieux)

-->