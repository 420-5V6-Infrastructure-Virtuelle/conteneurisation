---
title: 0 - Introduction à Docker
weight: 1010
---

## _Modularisez et maîtrisez vos applications_

![](../../../images/Moby-logo.png)

---

# Introduction

## La métaphore docker : "box it, ship it"

![](../../../images/docker/enVrac.jpg)

- Une abstraction qui ouvre de nouvelles possibilités pour la manipulation logicielle.
- Permet de standardiser, industrialiser et de contrôler la livraison et le déploiement.

# Retour sur les technologies de virtualisation

On compare souvent les conteneurs aux machines virtuelles. Mais ce sont de grosses simplifications parce qu'on en a un usage similaire : isoler des programmes dans des "contextes".
Une chose essentielle à retenir sur la différence technique : **les conteneurs utilisent les mécanismes internes du \_kernel de l'OS **Linux**\_ tandis que les VM tentent de communiquer avec l'OS (quel qu'il soit) pour directement avoir accès au matériel de l'ordinateur.**

<!-- ![](../../../images/hyperv-vs-containers.png) -->

![](../../../images/vm_vs_containers.png)

- **VM** : une abstraction complète pour simuler des machines

  - un processeur, mémoire, appels systèmes, carte réseau, carte graphique, etc.

- **conteneur** : un découpage dans Linux pour séparer des ressources (accès à des dossiers spécifiques sur le disque, accès réseau).

Les deux technologies peuvent utiliser un système de quotas pour l'accès aux ressources matérielles (accès en lecture/écriture sur le disque, sollicitation de la carte réseau, du processeur)

Si l'on cherche la définition d'un conteneur :

**C'est un groupe de _processus_ associé à un ensemble de permissions**.

L'imaginer comme une "boîte" est donc une allégorie un peu trompeuse, car ce n'est pas de la virtualisation (= isolation au niveau matériel).

---

# L'origine du Docker : concept du **conteneur**

Les conteneurs mettent en œuvre un vieux concept d'isolation des processus permis par la philosophie Unix du "tout est fichier".

Dans Unix, presque tout (processus, périphériques, sockets, mémoire, configuration) est exposé comme un fichier.

Grâce à cette abstraction, Unix a pu développer des mécanismes d’isolation des processus : chroot, permissions, namespaces, cgroups.

Les conteneurs modernes (Docker, LXC, Kubernetes) réutilisent ces mécanismes pour créer des environnements isolés, légers et reproductibles.

Les conteneurs sont une évolution moderne de ce concept : isoler ce qu’un processus peut “voir” du système.

Voici quelques un de ces concept d’isolation :

**1. chroot**

- Implémenté principalement par le programme `chroot` [*change root* : changer de racine], permet l'isolation du système de fichiers
  
  L'utilitaire chroot (change root) modifie le répertoire racine d'un processus en cours.
  - Rôle : Restreindre l'accès disque.
  - Fonctionnement : Le processus croit que le dossier assigné est la racine / du système.

**2. Les _namespaces_ (espaces de noms)**

- Les **_namespaces_**, un concept informatique pour parler simplement de…
  - groupes séparés auxquels on donne un nom, d'ensembles de choses sur lesquelles on colle une étiquette
  - on parle aussi de **contextes**
  - Les _namespaces_ sont inventés en 2002
  - popularisés lors de l'inclusion des 6 types de _namespaces_ dans le **noyau Linux** (3.8) en **2013**

- Les _namespaces_ correspondent à autant de types de **compartiments** nécessaires dans l'architecture Linux pour isoler des processus, il y 6 types de _namespaces_ :
  - PID : Isole les identifiants de processus (le conteneur possède son propre processus numéro 1).
  - NET : Fournit des interfaces réseau, des tables de routage et des ports indépendants.
  - MNT (Mount) : Permet de créer sont propre volume avec sont propre systèmes de fichiers.
  - IPC : isole la communication inter-processus entre les espaces de nommage.
  - UTS : Permet d'avoir un nom d'hôte (hostname).
  - USER : Isole l'utilisateur ID entre les namespace. Permet d'être root (UID 0) à l'intérieur du conteneur tout en étant un utilisateur standard sans privilèges sur l'hôte.

---

**3. Les _cgroups_ : derniers détails pour une vraie isolation**

- Après, il reste à s'occuper de limiter la capacité d'un conteneur à agir sur les ressources matérielles :

  - usage de la mémoire
  - du disque
  - du réseau
  - des appels système
  - du processeur (CPU)


Alors que les namespaces masquent ce que le processus peut voir, les cgroups restreignent ce que le processus peut consommer.
- Rôle : Répartition et contrôle du matériel.
- Fonctionnement : Le noyau applique des barrières physiques strictes sur un groupe de processus.
- Ressources bridées : Allocation du temps processeur (CPU), quantité maximale de mémoire vive (RAM), bande passante réseau et accès aux disques (I/O). Cela empêche un conteneur de saturer l'hôte (attaque par déni de service).

---


# Concept de la conteneurisation

**1. Isolation sans virtualisation lourde**

Grâce à l’idée que tout est fichier, on peut montrer à un processus une version limitée du système :

- un système de fichiers isolé
- un réseau isolé
- des processus isolés
- des ressources limitées

Un conteneur n’est pas une VM : c’est un processus isolé qui croit être seul.

**2. Reproductibilité et portabilité**

Comme tout est fichier :

- un conteneur = un ensemble de fichiers (image + configuration)
- on peut reconstruire exactement le même environnement sur n’importe quelle machine
- on peut versionner ces fichiers (Dockerfile, YAML Kubernetes)

Même environnement partout, même comportement partout.

**3. Sécurité et contrôle**

L’isolation des fichiers permet :

- de limiter ce qu’un conteneur peut lire ou écrire
- de restreindre son accès au réseau
- de contrôler ses ressources (CPU, RAM, I/O)

On maîtrise précisément ce que chaque conteneur peut faire.

En résumé, Le succès des conteneurs découle directement du design d'Unix, où chaque ressource (processus, réseau, disque) est représentée par un fichier. En isolant ce qu'un processus peut voir dans l'arborescence du système, on crée un environnement étanche et reproductible. Docker a modernisé ce concept en créant un format d'image standard, et Kubernetes s'occupe de déployer ces conteneurs sur des parcs de serveurs.

----------------------------------


# Les conteneurs : définition

On revient à notre définition d'un **conteneur** :

**Un conteneur est un groupe de _processus_ associé à un ensemble de permissions sur le système**.

> 1 container
> = 1 groupe de processus
>
> - des _namespaces_ (séparation entre ces groups)
> - des _cgroups_ (quota en ressources matérielles)

---

# LXC (LinuX Containers)

- En 2008 démarre le projet LXC qui chercher à rassembler les concepts d'isolation de processus de linux :

  - les **cgroups**
  - le **chroot**
  - les **namespaces**.

- Originellement, Docker était basé sur **LXC**. Il a depuis développé son propre assemblage de ces 3 mécanismes.

---

# Docker et LXC

- En 2013, Docker commence à proposer une meilleure finition et une interface simple qui facilite l'utilisation des conteneurs **LXC**.
- Puis il propose aussi son cloud, le **Docker Hub** pour faciliter la gestion d'images toutes faites de conteneurs.
- Au fur et à mesure, Docker abandonne le code de **LXC** (mais continue d'utiliser le **chroot**, les **cgroups** et **namespaces**).

- Le code de base de Docker (notamment **runC**) est open source : l'**Open Container Initiative** vise à standardiser et rendre robuste l'utilisation de containers.

---

# Avantages de la conteneurisation vs Virtualisation

Docker permet de faire des "quasi-machines" avec des performances proches du natif.

- Légèreté et performance : Les conteneurs consomment beaucoup moins de mémoire et d'espace disque car ils n'incluent pas de système d'exploitation invité (guest OS) complet.
- Démarrage instantané : Ils se lancent en quelques millisecondes (vs plusieurs minutes pour une machine virtuelle).
- Portabilité extrême : Tout le code et les dépendances sont packagés ensemble, garantissant que l'application fonctionne exactement de la même manière sur n'importe quel ordinateur ou serveur.
- Densité plus élevée : Il est possible d'exécuter de dix à cent fois plus de conteneurs que de machines virtuelles sur un même serveur physique.
- Moins **complexe** que la virtualisation
- Plus **standard** que les multiples hyperviseurs

---

# Avantages de la virtualisation vs Conteneurisation

- Isolation complète : Chaque machine virtuelle est totalement autonome. Si le système d'exploitation d'une VM est compromis, les autres restent en sécurité. L'isolation se fait au niveau du matériel et non au niveau du noyau de l'OS.
- Exécution de systèmes différents : La virtualisation permet de faire tourner simultanément des machines Linux, Windows ou macOS sur le même serveur physique.
- Gestion centralisée mature : Les outils de gestion d'infrastructure virtuelle (comme ceux de VMware ou Proxmox) offrent des fonctionnalités robustes et intégrées pour la sauvegarde, les snapshots et la haute disponibilité.

---

# Architecture qui combine les avantages des 2 concepts

L'exemple d'architecture le plus répandu dans l'industrie est le cluster Kubernetes déployé sur des Machines Virtuelles (VM). Et l'achitecture logiciel est en conteneur gérer par kubernetes.

---

# Pourquoi utiliser Docker ?

Docker est pensé dès le départ pour faire des **conteneurs applicatifs** :

- **isoler** les modules applicatifs.

- gérer les **dépendances** en les embarquant dans le conteneur.

- se baser sur l'**immutabilité** : la configuration d'un conteneur n'est pas faite pour être modifiée après sa création.

- avoir un **cycle de vie court** -> logique DevOps du "bétail vs. animal de compagnie"

---

# Pourquoi utiliser Docker ?

Docker modifie beaucoup la **"logistique"** applicative.

- **uniformisation** face aux divers langages de programmation, configurations et briques logicielles

- **installation sans accroc** et **automatisation** beaucoup plus facile

- permet de simplifier l'**intégration continue**, la **livraison continue** et le **déploiement continu** (CICD)

- **rapproche le monde du développement** des **opérations** (tout le monde utilise la même technologie) (DEVOPS)

- Permet l'adoption plus large de la logique DevOps (notamment le concept _d'infrastructure as code_)

---

# Infrastructure as Code

## Résumé

- on décrit en mode code un état du système. Avantages :
  - pas de dérive de la configuration et du système (immutabilité)
  - on peut connaître de façon fiable l'état des composants du système
  - on peut travailler en collaboration plus facilement (grâce à Git notamment)
  - on peut faire des tests
  - on facilite le déploiement de nouvelles instances

  Terraforme est devenu le logiciel symbole de l'IaC
---

# Docker : positionnement sur le marché

- Docker est la technologie ultra-dominante sur le marché de la conteneurisation

  - La simplicité d'usage et le travail de standardisation (un conteneur Docker est un conteneur OCI : format ouvert standardisé par l'Open Container Initiative) lui garantissent légitimité et fiabilité
  - La logique du conteneur fonctionne, et la bonne documentation et l'écosystème aident !

- **LXC** existe toujours et est très agréable à utiliser, notamment avec **LXD** (développé par Canonical, l'entreprise derrière Ubuntu) et **Proxmox**.

  - Il a cependant un positionnement différent : faire des conteneurs pour faire tourner des **OS Linux complets**.

- **Apache Mesos** : un logiciel de gestion de cluster qui permet de se passer de Docker, mais propose quand même un support pour les conteneurs OCI (Docker) depuis 2016.

- **Podman** : une alternative à Docker qui utilise la même syntaxe que Docker pour faire tourner des conteneurs OCI (Docker) qui propose un mode _rootless_ et _daemonless_ intéressant.(plus sécuritaire)

- **systemd-nspawn** : technologie de conteneurs isolés proposée par systemd

---
