---
title: Introduction à Docker
weight: 2010
---

## _Modularisez et maîtrisez vos applications_

![](../../../images/Moby-logo.png)

---

# Introduction

## La métaphore Docker : "box it, ship it"

![](../../../images/docker/enVrac.jpg)

- Une abstraction qui ouvre de nouvelles possibilités pour la manipulation logicielle.
- Permets de standardiser, industrialiser et de contrôler la livraison et le déploiement.

# Retour sur les technologies de virtualisation

On compare souvent les conteneurs aux machines virtuelles. Mais ce sont de grosses simplifications parce qu'on en a un usage similaire : isoler des programmes dans des "contextes".
Une chose essentielle à retenir sur la différence technique : **les conteneurs utilisent les mécanismes internes du \_kernel de l'OS **Linux**\_ tandis que les VM tentent de communiquer avec l'OS (quel qu'il soit) pour directement avoir accès au matériel de l'ordinateur.**

Docker Desktop utilise une machine virtuelle Linux pour exécuter les conteneurs sur Windows/macOS, car les conteneurs reposent sur des mécanismes du noyau Linux.

<!-- ![](../../../images/hyperv-vs-containers.png) -->

![](../../../images/vm_vs_containers.png)

- **Machine virtuelle - VM** : une abstraction complète pour simuler des machines
  - Isolation matérielle
  - Un processeur, mémoire, appels système, carte réseau, carte graphique, etc.
  - Tourne un système d’exploitation indépendant sur une même machine physique.
  - Agit comme un ordinateur autonome, avec son propre noyau, ses pilotes, ses services, etc.

- **Conteneur** : une isolation à l’échelle de l’application et il ne cherche pas à émuler un système entier.
  - Isolation logicielle
  - Vise à isoler uniquement l’application et ce dont elle a besoin pour fonctionner (code, dépendances, configs...)
  - Partagent le noyau du système hôte, ce qui les rend plus légers et plus rapides à exécuter.



---

# L'origine du Docker : concept du **conteneur**

Les conteneurs mettent en œuvre des concepts d'isolation des processus de Unix où "tout est fichier".

Dans Unix, le principe du “tout est fichier” fait que toutes les ressources du système, les processus, les périphériques, les sockets, la mémoire sont présentés comme des fichiers. 

Comme toutes les ressources du système sont représentées sous forme de fichiers, le noyau peut décider quels fichiers un processus voit. 

En contrôlant cette visibilité, Unix peut créer des environnements isolés où chaque processus perçoit une version limitée du système.

Les conteneurs modernes s’appuient sur les mécanismes d’isolation qui fournir à chaque processus une vision réduite, indépendante et reproductible du système.

**Voici quelques-uns de ces concepts d’isolation repris de Unix:**

**1. chroot**

- Implémenté principalement par le programme **_chroot_** [*change root* : changer de racine], permet l'isolation du système de fichiers

  - Redéfinit la racine du système de fichiers : chroot change le répertoire “/” visible par un processus. Le processus croit que le dossier assigné est la racine / du système.
    - le processus ne voit plus le vrai système de fichiers,
    - il ne peut pas toucher aux fichiers critiques du système,
    - il ne peut pas casser l’OS, même s’il se comporte mal.
  - Environnement isolé : le processus voit uniquement les fichiers présents dans ce nouveau “/”, comme s’il s’agissait d’un mini‑système.
  - Limitation de l’accès : le processus ne peut plus accéder aux fichiers en dehors de ce répertoire (sauf mauvaise configuration).
  - Utilisé pour tester ou réparer : permets de lancer des programmes dans un environnement contrôlé, utile pour du dépannage ou des installations.
  - Isolation partielle : contrairement aux namespaces, chroot n’isole que le système de fichiers, pas le réseau, les processus ou les utilisateurs.


**2. Les _namespaces_ (espaces de noms)**

- Les **_namespaces_** sont des mécanismes du noyau Linux qui créent des environnements isolés pour les processus

  - Isolation logique : un namespace crée une “bulle” où un processus voit une version limitée du système.
  - Ressources séparées : chaque namespace isole un type de ressource (PID, réseau, montage, utilisateur, etc.).
  - Vue indépendante : les processus dans un namespace ont leur propre vision des identifiants, des interfaces réseau, des points de montage, etc.
  - Non‑interférence : un processus dans un namespace ne peut pas voir ni affecter les ressources d’un autre namespace.
  - Contrôle du système: le noyau gère ces espaces isolés pour permettre une organisation fine et sécurisée des processus.
  - Les _namespaces_ sont inventés en 2002 et il existe 6 types de _namespaces_ dans le **noyau Linux** (3.8) en **2013**

- Ceux-ci correspondent à des **compartiments** nécessaires dans l'architecture Linux pour isoler des processus, il y 6 types de _namespaces_ :
  - **PID** : Isole les identifiants de processus (le conteneur possède son propre processus numéro 1).
  - **NET** : Fournis des interfaces réseau, des tables de routage et des ports indépendants.
  - **MNT** (Mount) : Permets de créer son propre volume avec son propre système de fichiers.
  - **IPC** : isole la communication interprocessus entre les espaces de nommage.
  - **UTS** : Permets d'avoir un nom d'hôte (hostname).
  - **USER** : Isole l'utilisateur ID entre les namespaces. Permets d'être root (UID 0) à l'intérieur du conteneur tout en étant un utilisateur standard sans privilèges sur l'hôte.

---

**3. Les _cgroups_ : derniers mécanismes pour un contrôle complet des ressources (CPU, mémoire, I/O)**

- Après, il reste à s'occuper de limiter la capacité à agir sur les ressources matérielles disponibles :

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

Un conteneur n’est pas une VM : c’est un ensemble de processus isolés qui croient être seuls et fonctionner dans leur propre environnement.

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

En résumé, le succès des conteneurs découle directement du design d'Unix, où chaque ressource (processus, réseau, disque) est représentée par un fichier. En isolant ce qu'un processus peut voir dans l'arborescence du système, on crée un environnement étanche et reproductible. Docker a modernisé ce concept en créant un format d'image standard, et Kubernetes s'occupe de déployer ces conteneurs sur des parcs de serveurs.

---

# Avantages de la conteneurisation vs Virtualisation

Docker permet de faire des "quasi-machines" avec des performances proches du natif.

- Légèreté et performance : Les conteneurs consomment beaucoup moins de mémoire et d'espace disque, car ils n'incluent pas de système d'exploitation invité (guest OS) complet.
- Démarrage instantané : Ils se lancent en quelques centaines de millisecondes à quelques secondes (vs plusieurs minutes pour une machine virtuelle).
- Portabilité extrême : Tout le code et les dépendances sont packagés ensemble, garantissant que l'application fonctionne exactement de la même manière sur n'importe quel ordinateur ou serveur.
- Densité plus élevée : Il est possible d'exécuter de dix à cent fois plus de conteneurs que de machines virtuelles sur un même serveur physique.
- Moins **complexe** que la virtualisation
- Plus **standard** que les multiples hyperviseurs

---

# Avantages de la virtualisation vs Conteneurisation

- Isolation complète : Chaque machine virtuelle est totalement autonome. Si le système d'exploitation d'une VM est compromis, les autres restent en sécurité. L'isolation se fait au niveau du matériel et non au niveau du noyau de l'OS.
- Exécution de systèmes différents : La virtualisation permet de faire tourner simultanément des machines Linux, Windows ou macOS sur le même serveur physique.
- Gestion centralisée mature : Les outils de gestion d'infrastructure virtuelle (comme ceux de VMware ou Proxmox) offrent des fonctionnalités robustes et intégrées pour la sauvegarde, les snapshots et la haute disponibilité.



#### Architecture qui combine les avantages des 2 concepts

L'exemple d'architecture le plus répandu dans l'industrie est le cluster Kubernetes déployé sur des Machines virtuelles (VM). Et l'architecture logicielle est en conteneur géré par Kubernetes.

Kubernetes peut fonctionner sur des serveurs bare‑metal, mais dans la majorité des environnements professionnels, il est déployé sur des machines virtuelles pour des raisons de gestion, de sécurité et de flexibilité.

#### Résumé des avantages des deux technologies

| Critère | Virtualisation (VMs) | Conteneurisation |
| :--- | :--- | :--- |
| **Isolation** | OS complet isolé | Processus isolé, noyau partagé |
| **Taille** | Plusieurs Go | Quelques Mo à centaines de Mo |
| **Démarrage** | Minutes | Secondes |
| **Ressources** | Élevées (RAM, CPU dédiés) | Légères (partage des ressources) |
| **Portabilité** | Dépends de l'hyperviseur | Très portable (image OCI standard) |
| **Cas d'usage** | Multi-OS, isolation forte | Microservices, CI/CD, cloud-native |

---

# Pourquoi utiliser Docker ?

#### Docker est pensé dès le départ pour faire des **conteneurs applicatifs** :

- **isoler** les modules applicatifs.

- gérer les **dépendances** en les embarquant dans le conteneur.

- se baser sur l'**immutabilité** : la configuration d'un conteneur n'est pas faite pour être modifiée après sa création.

- avoir un **cycle de vie court** -> logique DevOps du "bétail vs. animal de compagnie"

#### Docker modifie beaucoup la **"logistique"** applicative.

- **uniformisation** face aux divers langages de programmation, configurations et briques logicielles

- **installation sans accroc** et **automatisation** beaucoup plus facile

- permet de simplifier l'**intégration continue**, la **livraison continue** et le **déploiement continu** (CICD)

- **rapproche le monde du développement** des **opérations** (tout le monde utilise la même technologie) (DEVOPS)

- Permets l'adoption plus large de la logique DevOps (notamment le concept _d'infrastructure as code_)

---

# Un peu d'histoire

#### LXC (LinuX Containers)

- En 2008, démarre le projet LXC qui cherche à rassembler les concepts d'isolation de processus de Linux :

  - les **cgroups**
  - le **chroot**
  - les **namespaces**.

- Originellement, Docker était basé sur **LXC**. Il a depuis développé son propre assemblage de ces 3 mécanismes.


#### Docker et LXC

- En 2013, Docker commence à proposer une meilleure finition et une interface simple qui facilite l'utilisation des conteneurs **LXC**.
- Puis il propose aussi son cloud, le **Docker Hub** pour faciliter la gestion d'images toutes faites de conteneurs.
- Au fur et à mesure, Docker abandonne le code de **LXC** (mais continue d'utiliser le **chroot**, les **cgroups** et **namespaces**).

- Le code de base de Docker (notamment **runC**) est open source : l'**Open Container Initiative** vise à standardiser et rendre robuste l'utilisation de containers.

---

# Docker : positionnement sur le marché

**Docker** est la technologie ultra-dominante sur le marché de la conteneurisation

  - La simplicité d'usage et le travail de standardisation (un conteneur Docker est un conteneur OCI : format ouvert standardisé par l'Open Container Initiative) lui garantissent légitimité et fiabilité
  - La logique du conteneur fonctionne, et la bonne documentation et l'écosystème aident !

**LXC** existe toujours et est très agréable à utiliser, notamment avec **LXD** (développé par Canonical, l'entreprise derrière Ubuntu) et **Proxmox**.

  - Il a cependant un positionnement différent : faire des conteneurs pour faire tourner des **OS Linux complets**.
    - LXC/LXD vise à exécuter des conteneurs système (des environnements Linux complets), tandis que Docker se concentre sur des conteneurs applicatifs conçus pour isoler et exécuter une application avec ses dépendances.

**Apache Mesos** : un logiciel de gestion de cluster qui permet de se passer de Docker, mais propose quand même un support pour les conteneurs OCI (Docker) depuis 2016.

**Podman** : une alternative à Docker qui utilise la même syntaxe que Docker pour faire tourner des conteneurs OCI (Docker) qui proposent un mode _rootless_ et _daemonless_ intéressants.(plus sécuritaire)

**systemd-nspawn** : technologie de conteneurs isolés proposée par systemd

---

# À retenir

  - **Conteneurs vs VMs** : les conteneurs partagent le noyau de l’hôte et sont très légers; les VMs embarquent un OS complet et offrent une isolation matérielle plus forte.
  - **Isolation Linux** : chroot isole le système de fichiers, les namespaces isolent ce qu’un processus peut voir, les cgroups limitent ce qu’il peut consommer.
  - **Philosophie Unix** : “tout est fichier” : Comme toutes les ressources sont représentées comme des fichiers, le noyau peut contrôler la visibilité et créer des environnements isolés.
  - **Conteneur = environnement minimal** : Un conteneur est un ensemble de processus isolés qui croient fonctionner dans leur propre système.
  - **Légèreté et rapidité** : Les conteneurs démarrent en quelques millisecondes à secondes, consomment peu de ressources et sont hautement portables.

---