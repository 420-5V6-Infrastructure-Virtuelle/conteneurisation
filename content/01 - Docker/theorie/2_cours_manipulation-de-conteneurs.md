---
title: Utiliser les commandes Docker
weight: 1012
---

# Les images et conteneurs


![](../../../images/docker-cycle.jpg)

Docker fonctionne avec un CLI et propose de grandes quantités d'options pour chaque commande.

**Docker** possède à la fois un module pour lancer les applications (runtime) et un **outil de build** d'application.

- Une image est le **résultat** d'un build :
  - on peut la voir un peu comme un "modèle" de conteneur, nous allons voir plus loin comment "builder une image"

Pour lister les images on utilise :

```bash
docker images
docker image ls
```

---

## Les conteneurs

```bash
docker image --help
```

Utilisez `--help` après chaque commande, sous-commande ou sous-sous-commandes pour voir les options et possiblités

---

# Pour vérifier l'état de Docker

- Les commandes de base pour connaître l'état de Docker sont :

```bash
docker info  # affiche plein d'information sur l'engine avec lequel vous êtes en contact
docker ps    # affiche les conteneurs en train de tourner
docker ps -a # affiche  également les conteneurs arrêtés
```

### Créer et lancer un conteneur

![](../../../images/ops-basics-isolation.svg)

```bash
docker run [-d] [-p port_h:port_c] [-v dossier_h:dossier_c] <image> <commande>
```

> créé et lance le conteneur

- **L'ordre des arguments est important !**
- **Un nom est automatiquement généré pour le conteneur à moins de fixer le nom avec `--name`**
- On peut facilement lancer autant d'instances que nécessaire tant qu'il n'y a **pas de collision** de **nom** ou de **port**.

---

### Options docker run

- Les options facultatives indiquées ici sont très courantes.
  - `-d` permet\* de lancer le conteneur en mode **daemon** ou **détaché** et libérer le terminal
  - `-p` permet de mapper un _port réseau_ entre l'intérieur et l'extérieur du conteneur, typiquement lorsqu'on veut accéder à l'application depuis l'hôte.
  - `-v` permet de monter un _volume_ partagé entre l'hôte et le conteneur.
  - `--rm` (comme _remove_) permet de supprimer le conteneur dès qu'il s'arrête.
  - `-it` permet de lancer une commande en mode _interactif_ (un terminal comme `bash`).
  - `-a` (ou `--attach`) permet de se connecter à l'entrée-sortie du processus dans le container.

---

## Commandes Docker

- Le démarrage d'un conteneur est lié à une **commande**.

- Si le conteneur n'a pas de commande, il s'arrête dès qu'il a fini de démarrer

```bash
docker run debian # s'arrête tout de suite
```

- Pour utiliser une commande on peut simplement l'ajouter à la fin de la commande run.

```bash
docker run debian echo 'attendre 10s' && sleep 10 # s'arrête après 10s
```

---

### Stopper et redémarrer un conteneur

`docker run` créé un nouveau conteneur à chaque fois.

```bash
docker stop <nom_ou_id_conteneur> # ne détruit pas le conteneur
docker start <nom_ou_id_conteneur> # le conteneur a déjà été créé
docker start --attach <nom_ou_id_conteneur> # lance le conteneur et s'attache à la sortie standard
```

---



# Introspection de conteneur

- La commande `docker exec` permet d'exécuter une commande à l'intérieur du conteneur **s'il est lancé**.

- Une utilisation typique est d'introspecter un conteneur en lançant `bash` (ou `sh`).

```
docker exec -it <conteneur> /bin/bash
```

---

# Docker Hub : télécharger des images

Une des forces de Docker vient de la distribution d'images :

- pas besoin de dépendances, on récupère une boîte autonome

- pas besoin de multiples versions en fonction des OS

Dans ce contexte un élément qui a fait le succès de Docker est le Docker Hub : [hub.docker.com](https://hub.docker.com)

Il s'agit d'un répertoire public et souvent gratuit d'images (officielles ou non) pour des milliers d'applications pré-configurées.

---

# Docker Hub:

- On peut y chercher et trouver presque n'importe quel logiciel au format d'image Docker.

- Il suffit pour cela de chercher l'identifiant et la version de l'image désirée.

- Puis utiliser `docker run [<compte>/]<id_image>:<version>`

- La partie `compte` est le compte de la personne qui a poussé ses images sur le Docker Hub. Les images Docker officielles (`ubuntu` par exemple) ne sont pas liées à un compte : on peut écrire simplement `ubuntu:focal`.

- On peut aussi juste télécharger l'image : `docker pull <image>`

Exemple :

```bash
docker pull nginx
```

```bash
docker pull nginx:1.21.0
```

On peut également y créer un compte gratuit pour pousser et distribuer ses propres images, ou installer son propre serveur de distribution d'images privé ou public, appelé **registry**.

---

# En résumé

![](../../../images/docker-architecture.png)

<!-- ![](../../../images/docker-components.png) -->
