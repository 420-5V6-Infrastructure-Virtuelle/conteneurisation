---
title: Volumes
weight: 2060
---




Les conteneurs Docker sont, par conception, éphémères : lorsqu’un conteneur est supprimé, tout son système de fichiers disparaît avec lui. 

Pour éviter cette perte de données, Docker propose un mécanisme de persistance indépendant du cycle de vie des conteneurs : les volumes.

Trois solutions permettent de gérer la persistance selon le niveau de contrôle souhaité local et distant
  - Sur la machine hôte :
    - Volumes Docker (standard) : stockés et gérés automatiquement par Docker, idéals pour la portabilité et la simplicité.
    - Bind mounts : Accès direct à un répertoire du système de fichiers de l’hôte, utile pour le développement ou l’intégration avec des outils externes.
    - tmpfs : Stockage en mémoire RAM, parfait pour les données sensibles ou temporaires nécessitant des performances maximales.
  - Sur le réseau
    - Volumes NFS (Network File System) : partage de données entre plusieurs hôtes Docker
    - CIFS/SMB : partage Windows/Samba
    - Plugins de volumes : extension avec SSHFS et autres solutions cloud (AWS EBS, Azure Disk, etc.)
</br> </br>


#### Tableau comparatif des solutions locales
</br>

| Critère            | Volume standard               | Bind Mount                 | tmpfs              |
|--------------------|-------------------------------|----------------------------|--------------------|
| **Stockage**       | /var/lib/docker/volumes/      | Chemin hôte quelconque     | RAM uniquement     |
| **Persistance**    | ✓ Oui                         | ✓ Oui                      | ✗ Non (perdu à l'arrêt) |
| **Géré par Docker**| ✓ Oui                         | ✗ Non                      | ✗ Non              |
| **Performance**    | Bonne                         | Excellente (accès direct)  | Ultrarapide       |
| **Sécurité**       | ✓ Isolé                       | ⚠️ Expose l'hôte           | ✓ Jamais sur disque |
| **Portabilité**    | ✓ Facile à migrer             | ✗ Dépends de l'hôte         | N/A                |
| **Cas d’usage**    | Production, BDD               | Développement, hot-reload  | Secrets, cache, sessions |


#### Les Volumes Standards

Ce type de volumes représentent la méthode recommandée par Docker pour la persistance des données. Docker gère l'emplacement, la sécurité et le cycle de vie de vos données.

Il offre une bonne performance, une portabilité élevée et constitue l’option recommandée pour la production, notamment pour les bases de données et les applications nécessitant des données durables.

Le principe de ce volume est que l'espace de stockage est dédié, créé et géré par Docker dans un répertoire spécial (/var/lib/docker/volumes/). 

Contrairement aux fichiers du conteneur, le volume survit à la suppression du conteneur.

##### Création et utilisation

```bash
# Création du volume
docker volume create mon_volume

# Utilisation par un conteneur
# mon_volume est monté à l'emplacement /path/in/container dans le conteneur.
docker run -v mon_volume:/path/in/container ...

# On peut utiliser --mount au lieu de -v
# --mount : plus lisible, échoue avec une erreur claire si le chemin source n'existe pas (bind mount), indispensable pour les options avancées
# exemple : 
docker run --mount type=volume,source=mon_volume,target=/app/data nginx
```

#### Bind Mount : 

C'est un accès direct au système de fichiers de l'hôte (host) et le mount est sur un dossier de votre machine vers un dossier dans le conteneur. 

C'est l'outil préféré des développeurs pour le hot-reload : modifiez votre code, le conteneur voit immédiatement les changements.

Avantages des Bind Mounts - Utilisé par les développeurs
- Hot-reload : modifications visibles instantanément, idéal pour le développement
- Performances : accès direct au disque, pas de couche d'abstraction Docker
- Débogage : inspectez les fichiers générés par le conteneur depuis l'hôte

##### Création et utilisation
```bash
# Syntaxe -v (chemin absolu obligatoire !)
docker run -v /home/user/projet:/app nginx

# Syntaxe --mount (plus explicite)
docker run --mount type=bind,source=/home/user/projet,target=/app nginx

# Exemple que les développeurs utilisent
# Le code local est directement accessible dans le conteneur
docker run \
  # Monte le dossier src de l’hôte dans /app/src du conteneur
  --mount type=bind,source="$(pwd)/src",target=/app/src \
  # Monte le fichier package.json de l’hôte dans /app/package.json du conteneur
  --mount type=bind,source="$(pwd)/package.json",target=/app/package.json \
  # Image utilisée pour exécuter l’application
  node:20 \
  # Commande exécutée à l’intérieur du conteneur
  npm run dev
```

#### tmpfs 
Le montage tmpfs stocke les données directement en RAM (mémoire vive). Ce qui donne des performances fulgurantes, mais les données disparaît à l'arrêt du conteneur. 

C'est parfait pour les secrets, le cache, ou les fichiers temporaires. C'est un stockage ultrarapide, mais éphémère.

Avantage du tmpfs
  - Sécurité : les données sensibles (tokens, mots de passe temporaires) ne sont jamais écrites sur disque
  - Performance : la RAM est 10 à 100 fois plus rapide qu'un SSD

##### Création et utilisation

```bash
# Syntaxe simple
docker run --tmpfs /app/tmp nginx

# tmpfs avec limite de taille (100 Mo max)
docker run --mount type=tmpfs,destination=/app/tmp,tmpfs-size=100m nginx

# tmpfs avec permissions spécifiques (mode 1777 = sticky bit + rwx pour tous)
docker run --mount type=tmpfs,destination=/tmp,tmpfs-mode=1777 nginx

# Combinaison taille + mode
docker run --mount type=tmpfs,destination=/app/cache,tmpfs-size=200m,tmpfs-mode=1755 nginx
```

### Volumes Réseau : NFS et CIFS

Les volumes réseau permettent à Docker de stocker des données hors de l’hôte local, sur un serveur externe. 

On les monte via --mount en spécifiant un driver comme NFS ou CIFS/SMB. 

Ils offrent une persistance centralisée, partage entre conteneurs et hôtes, et conviennent aux environnements distribués ou multiserveurs.

Quand utiliser les volumes réseau ?
  - Clusters Docker avec plusieurs nœuds (Swarm, standalone)
  - Données partagées entre applications sur différents serveurs
  - Stockage centralisé sur NAS (Synology, QNAP, TrueNAS)
  - Migration de conteneurs entre hôtes sans perte de données

#### Volumes NFS (Network File System)

NFS est le protocole standard pour le partage de fichiers entre systèmes Linux. Il offre d'excellentes performances et une configuration relativement simple.

Prérequis NFS
  - Un serveur NFS correctement configuré et accessible depuis l'hôte Docker
  - Le client NFS installé sur l'hôte Docker

##### Création et utilisation

```bash
# Installation du client NFS sur l'hôte Docker
sudo apt install nfs-common

# Création d'un Volume NFS
docker volume create --driver local \
  --opt type=nfs \
  --opt o=addr=192.168.1.50,nolock,soft,rw \
  --opt device=:/exports/data \
  mon_volume_nfs

# Vérifier que le volume existe
docker volume ls

# Utiliser dans un conteneur
docker run -v mon_volume_nfs:/data nginx
```

##### Volumes CIFS/SMB (Windows/Samba

CIFS (Common Internet File System), aussi appelé SMB (Server Message Block), est le protocole de partage de fichiers de Windows. 

Il est idéal pour intégrer Docker avec des serveurs Windows ou des NAS compatibles Samba.


Prérequis CIFS/SMB (Windows/Samba)

Le package cifs-utils doit être installé sur l'hôte Docker

##### Création et utilisation

```bash
# Installation du client CIFS sur l'hôte Docker
sudo apt install cifs-utils

# Création d'un Volume NFS
docker volume create --driver local \
  --opt type=cifs \
  --opt o=addr=192.168.1.100,username=utilisateur,password=motdepasse,vers=3.0 \
  --opt device=//192.168.1.100/partage \
  mon_volume_cifs

# Vérifier que le volume existe
docker volume ls

# Utiliser dans un conteneur
docker run -v mon_volume_cifs:/data nginx
```

Sécuriser les identifiants CIFS
```bash
# Créer un fichier sécurisé
echo "username=utilisateur" > /etc/docker-cifs-credentials
echo "password=motdepasse" >> /etc/docker-cifs-credentials
chmod 600 /etc/docker-cifs-credentials

# Créer le volume avec le fichier d'identifiants
docker volume create --driver local \
  --opt type=cifs \
  --opt o=credentials=/etc/docker-cifs-credentials,vers=3.0 \
  --opt device=//serveur/partage \
  mon_volume_cifs_secure
```

<!-- Ajout schéma -->
<!-- Ajout raisonnement tout ce qui est stateful sur un volume : fichiers de config, certifs, fichiers de base de données -->

## Les volumes Docker via la sous-commande `volume`

- `docker volume ls`
- `docker volume inspect`
- `docker volume prune`
- `docker volume create`
- `docker volume rm`

<!-- ## Volumes nommés -->
<!-- Où sont-ils stockés -->

## Bind mounting

Lorsqu'un répertoire hôte spécifique est utilisé dans un volume (la syntaxe `-v HOST_DIR:CONTAINER_DIR`), elle est souvent appelée **bind mounting** ("montage lié").
C'est quelque peu trompeur, car tous les volumes sont techniquement "bind mounted". La particularité, c'est que le point de montage sur l'hôte est explicite plutôt que caché dans un répertoire appartenant à Docker.

Exemple :

```bash
# Sur l'hôte
docker run -it -v /home/user/app/config.conf:/config/main.conf:ro -v /home/user/app/data:/data ubuntu /bin/bash

# Dans le conteneur
cd /data/
touch testfile
exit

# Sur l'hôte
ls /home/user/app/data:
```

## Volumes nommés

- L'autre technique est de créer d'abord un volume nommé avec :
  `docker volume create mon_volume`
  `docker run -d -v mon_volume:/data redis`

---

### L'instruction `VOLUME` dans un `Dockerfile`

L'instruction `VOLUME` dans un `Dockerfile` permet de désigner les volumes qui devront être créés lors du lancement du conteneur. On précise ensuite avec l'option `-v` de `docker run` à quoi connecter ces volumes. Si on ne le précise pas, Docker crée quand même un volume Docker au nom généré aléatoirement, un volume "caché".



### Partager des données avec un volume

- Pour partager des données, on peut monter le même volume dans plusieurs conteneurs.

- Pour lancer un conteneur avec les volumes d'un autre conteneur déjà montés on peut utiliser `--volumes-from <container>`

- On peut aussi créer le volume à l'avance et l'attacher après coup à un conteneur.

- Par défaut le driver de volume est `local` c'est-à-dire qu'un dossier est créé sur le disque de l'hôte.

```bash
docker volume create --driver local \
    --opt type=btrfs \
    --opt device=/dev/sda2 \
    monVolume
```

---

### Plugins de volumes

On peut utiliser d'autres systèmes de stockage en installant de nouveaux plugins de driver de volume. Par exemple, le plugin `vieux/sshfs` permet de piloter un volume distant via SSH.

Exemples:

- SSHFS (utilisation d'un dossier distant via SSH)
- NFS (protocole NFS)
- BeeGFS (système de fichier distribué générique)
- Amazon EBS (vendor specific)
- etc.

```bash
docker volume create -d vieux/sshfs -o sshcmd=<sshcmd> -o allow_other sshvolume
docker run -p 8080:8080 -v sshvolume:/path/to/folder --name test someimage
```

---

Ou via docker-compose :

```yaml
volumes:
  sshfsdata:
    driver: vieux/sshfs:latest
    driver_opts:
      sshcmd: "username@server:/location/on/the/server"
      allow_other: ""
```

---

### Permissions

- Un volume est créé avec les permissions du dossier préexistant.

```Dockerfile
FROM debian
RUN groupadd -r graphite && useradd -r -g graphite graphite
RUN mkdir -p /data/graphite && chown -R graphite:graphite /data/graphite
VOLUME /data/graphite
USER graphite
CMD ["echo", "Data container for graphite"]
```

---

### Backups de volumes

- Pour effectuer une sauvegarde, la méthode recommandée est d'utiliser un conteneur supplémentaire dédié
- Qui accède au volume avec `--volume-from`
- Qui est identique aux autres et donc normalement avec les mêmes UID/GID/permissions.
<!-- - permet de ne pas perdre bêtement le volume lors d'un `prune` car il reste un conteneur qui y est lié -->
