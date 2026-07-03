---
title: 0 - Introduction à Docker, pour préparation enseignant
weight: 1010
draft: true
---

Regarder la vidéo pour comprendre l'évolution de la contenerisation et de l'isolation de processus de unix jusqu'à Docker.
https://www.youtube.com/watch?v=dikQOyAzdS4

# Docker Origins : genèse du concept de **conteneur**

Les conteneurs mettent en œuvre un vieux concept d'isolation des processus permis par la philosophie Unix du "tout est fichier".
--------------------------------

Dans Unix, presque tout (processus, périphériques, sockets, mémoire, configuration) est exposé comme un fichier.

Grâce à cette abstraction, Unix a pu développer des mécanismes d’isolation des processus : chroot, permissions, namespaces, cgroups.

Les conteneurs modernes (Docker, LXC, Kubernetes) réutilisent ces mécanismes pour créer des environnements isolés, légers et reproductibles.

Les conteneurs sont une évolution moderne de ce concept : isoler ce qu’un processus peut “voir” du système.


## `chroot`, `jail`, les 6 `namespaces` et les `cgroups`

### 1. chroot 

- Implémenté principalement par le programme `chroot` [*change root* : changer de racine], présent dans les systèmes UNIX depuis longtemps (1979 !) :
  > "Comme tout est fichier, changer la racine d'un processus, c'est comme le faire changer de système".

  L'isolation du système de fichiers
  
  L'utilitaire chroot (change root) modifie le répertoire racine d'un processus en cours.
  - Rôle : Restreindre l'accès disque.
  - Fonctionnement : Le processus croit que le dossier assigné est la racine / du système.
  - Limite : Il ne s'agit pas d'une barrière de sécurité étanche. Un utilisateur root à l'intérieur d'un environnement chroot (souvent appelé chroot jail) peut facilement en "s'échapper" via des appels système spécifiques. De plus, il voit toujours les autres processus de la machine.

### 2. jail

- `jail` est introduit par FreeBSD en 2002 pour compléter `chroot` et qui permet pour la première fois une **isolation réelle (et sécurisée) des processus**.
- `chroot` ne s'occupait que de l'isolation d'un process par rapport au système de fichiers :

  - ce n'était pas suffisant, l'idée de "tout-est-fichier" possède en réalité plusieurs exceptions
  - un process _chrooté_ n'est pas isolé du reste des process et peut agir de façon non contrôlée sur le système sur plusieurs aspects
    <!-- - expliquer chroot: notamment démo de comment on en échappe ? -->

- En 2005, Sun introduit les **conteneurs Solaris** décrits comme un « chroot sous stéroïdes » : comme les _jails_ de FreeBSD

### 3.  Les _namespaces_ (espaces de noms)

- Les **_namespaces_**, un concept informatique pour parler simplement de…
  - groupes séparés auxquels on donne un nom, d'ensembles de choses sur lesquelles on colle une étiquette
  - on parle aussi de **contextes**
- `jail` était une façon de _compléter_ `chroot`, pour FreeBSD.
- Pour Linux, ce concept est repris via la mise en place de **namespaces Linux**

  - Les _namespaces_ sont inventés en 2002
  - popularisés lors de l'inclusion des 6 types de _namespaces_ dans le **noyau Linux** (3.8) en **2013**

- Les conteneurs ne sont finalement que **plein de fonctionnalités Linux ensemblé de façon cohérente**.
- Les _namespaces_ correspondent à autant de types de **compartiments** nécessaires dans l'architecture Linux pour isoler des processus.

Pour la culture, 6 types de _namespaces_ :

- PID : Isole les identifiants de processus (le conteneur possède son propre processus numéro 1).
- NET : Fournit des interfaces réseau, des tables de routage et des ports indépendants.
- MNT (Mount) : Permet de créer sont propre volume avec sont propre systèmes de fichiers.
- IPC : isole la communication inter-processus entre les espaces de nommage.
- UTS : Permet d'avoir un nom d'hôte (hostname).
- USER : Isole l'utilisateur ID entre les namespace. Permet d'être root (UID 0) à l'intérieur du conteneur tout en étant un utilisateur standard sans privilèges sur l'hôte.

---

### 3. Les _cgroups_ : derniers détails pour une vraie isolation

- Après, il reste à s'occuper de limiter la capacité d'un conteneur à agir sur les ressources matérielles :

  - usage de la mémoire
  - du disque
  - du réseau
  - des appels système
  - du processeur (CPU)

La limitation des ressources physiques

Alors que les namespaces masquent ce que le processus peut voir, les cgroups restreignent ce que le processus peut consommer.
- Rôle : Répartition et contrôle du matériel.
- Fonctionnement : Le noyau applique des barrières physiques strictes sur un groupe de processus.
- Ressources bridées : Allocation du temps processeur (CPU), quantité maximale de mémoire vive (RAM), bande passante réseau et accès aux disques (I/O). Cela empêche un conteneur de saturer l'hôte (attaque par déni de service).

- En 2005, Google commence le développement des **cgroups** : une façon de _tagger_ les demandes de processeur et les appels systèmes pour les grouper et les isoler.

---

