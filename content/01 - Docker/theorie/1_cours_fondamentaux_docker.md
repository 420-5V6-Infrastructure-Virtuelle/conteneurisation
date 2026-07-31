---
title: Les concepts fondamentaux
weight: 2020
---

# Terminologie et concepts fondamentaux

Deux concepts centraux :

Une **image** :  
  - C’est comme une photo instantanée d'une'application : un ensemble figé, immuable et prêt à être exécuté dans un environnement isolé. 
  - Elle embarque tout ce dont ton application a besoin pour tourner correctement : le code, les dépendances, les variables d’environnement, les fichiers de configuration, etc.
  - C'est un modèle pour créer un conteneur
  - N'est pas une instance vivante, mais une recette pour faire tourner une application

Un **conteneur** : 
  - Un conteneur, c’est l’image en cours d’exécution
  - C'est une instance vivante, un processus isolé basé sur l’image qui tourne sur la machine.
  - Il est de nature **éphémère,** car il vit le temps où il tourne : dès qu’il est supprimé, tout son système de fichiers et ses données disparaissent avec lui.


Autres concepts primordiaux :

Un **volume** : 
  - Espace virtuel pour gérer le stockage d'un conteneur et le partage entre conteneurs.
  - Stockage persistant qui vit en dehors du conteneur
  - Conserver des données même si le conteneur (éphémère) est supprimé, recréé ou mis à jour.

Un **réseau** : Les conteneurs Docker sont isolés par défaut : sans configuration réseau explicite, ils ne peuvent ni communiquer entre eux ni accéder à l’extérieur.
- Il y a 6 types de réseau
  - Bridge : le réseau par défaut, idéal pour connecter des conteneurs sur un même hôte
  - Host : performances maximales en partageant le réseau de l'hôte
  - Overlay : communication entre conteneurs sur différents serveurs (Swarm/Kubernetes)
  - Macvlan et Ipvlan : intégration directe au réseau physique (applications legacy)
  - None : désactive simplement la pile réseau du conteneur.


Un **registre d'image de conteneur** : 
  - C'est un serveur où sont stocké et distribue des images versionnées
    - Un peu comme GitHub pour le code
  - Docker Hub est le registry le plus connu d'images officielles et communautaires accessibles sur le web.
  - JFrog Artifactory, Nexus repository et Harbor sont des solutions de registries pour entreprise.


---

# L'écosystème  Docker

Pour cette partie du cours, nous allons utiliser Docker comme plateforme de conteneurisation. Il existe plusieurs autres solutions, mais Docker demeure la référence du secteur.

Il permet d’emballer une application et toutes ses dépendances dans une image, puis de l’exécuter de manière uniforme sur n’importe quel **système**. 

Docker desktop s'installe sur tous les OS actuels, ce qui permet d'élimine le fameux « ça marche sur ma machine », puisque l’environnement d’exécution est entièrement standardisé et autonome.

# Architecture de Docker 

Docker repose sur une architecture client‑serveur. Le client Docker envoie des commandes au démon Docker, qui se charge de tout le travail :
  - construire les images 
  - lancer les conteneurs 
  - gérer les volumes 
  - Les réseaux et distribuer les artefacts.

Le client et le démon peuvent fonctionner sur la même machine, ou le client peut se connecter à un démon Docker distant.

La communication entre les deux se fait via une API REST, accessible par un socket UNIX ou une interface réseau.

Docker Compose est un autre client : il permet de gérer des applications composées de plusieurs conteneurs.

# Schéma d’architecture Docker

![](../../../images/docker/architecture_docker.jpg)


# Le Docker daemon (dockerd)

Le démon Docker (dockerd) écoute les requêtes envoyées via l’API Docker et gère tous les objets Docker :
- images
- conteneurs
- réseaux
- volumes


# Le Docker client (Docker)

Le client Docker (Docker) est l’outil principal utilisé par les développeurs et administrateurs.
Quand tu exécutes une commande comme Docker run, le client envoie l’instruction au démon, qui l’exécute.
Le client utilise l’API Docker et peut communiquer avec plusieurs démons simultanément.

# Docker Desktop

Docker Desktop est une application simple à installer pour macOS, Windows et certaines distributions Linux.
Elle regroupe tout ce qu’il faut pour développer et exécuter des applications conteneurisées :
- le démon Docker (dockerd)
- le client Docker (Docker)
- Docker Compose
- Docker Content Trust
- Kubernetes (optionnel)
- Credential Helper

C’est l’environnement le plus pratique pour les étudiants et les développeurs qui travaillent sur poste de travail.

# Les registres d’images Docker

Un registre est un service qui stocke et distribue des images de conteneurs.

Docker Hub est le registre public le plus connu, utilisé par défaut par Docker.

Il est aussi possible d’héberger un registre privé pour une organisation.

Lorsqu’on exécute Docker pull ou Docker run, Docker télécharge automatiquement l’image depuis le registre configuré.
Lorsqu’on exécute Docker push, Docker envoie l’image vers ce registre.

# Conteneur sur Docker
un conteneur Docker en cours de fonctionnement est un processus (et ses processus enfants) qui tourne dans une machine Linux hôte (mais ce processus est isolé des processus de l'hôte)
    - La **grande majorité** des conteneurs tournent sur un **noyau Linux**.
    - Sur des host Windows/macOS, Docker lance une VM Linux pour faire tourner les conteneurs. (Souvent WSL2 pour windows)
    - Il existe aussi des conteneurs qui roulent sur un noyau Windows, beaucoup moins répandus, surtout utilisés pour des applications .NET Framework legacy