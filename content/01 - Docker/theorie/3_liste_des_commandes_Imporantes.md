---
title: Liste des commandes imp.
weight: 2040
---

### Gestion des Images

| Action | Commande | Description |
| :--- | :--- | :--- |
| **Lister** | `docker images` | Affiche les images téléchargées localement |
| **Télécharger** | `docker pull <image>` | Télécharge une image depuis Docker Hub |
| **Construire** | `docker build -t <nom> .` | Construis une image via le fichier Dockerfile |
| **Supprimer** | `docker rmi <nom/id>` | Supprime une image locale |

### Gestion des Conteneurs

| Action | Commande | Description |
| :--- | :--- | :--- |
| **Créer et démarrer** | `docker run -d -p port:port image` | Lance un conteneur en arrière-plan (-d) avec mappage de port |
| **Lister (actifs)** | `docker ps` | Affiche les conteneurs en cours d'exécution |
| **Lister (tous)** | `docker ps -a` | Affiche tous les conteneurs (même arrêtés) |
| **Arrêter** | `docker stop <nom/id>` | Stoppe un conteneur proprement |
| **Démarrer** | `docker start <nom/id>` | Démarre un conteneur existant |
| **Supprimer** | `docker rm <nom/id>` | Supprime un conteneur arrêté |

### Diagnostic et Débogage

| Action | Commande | Description |
| :--- | :--- | :--- |
| **Logs en temps réel** | `docker logs -f <nom/id>` | Suit les sorties console du conteneur en direct |
| **Logs récents** | `docker logs --tail 50 <nom/id>` | Affiche uniquement les 50 dernières lignes de logs |
| **Logs horodatés** | `docker logs --since 30m <nom/id>` | Affiche les logs des 30 dernières minutes |
| **Accéder au shell** | `docker exec -it <nom/id> bash` | Ouvre un terminal interactif (utiliser `sh` si bash est absent) |
| **Déboguer un crash** | `docker run -it --entrypoint sh <image>` | Force le démarrage d'une image avec un shell pour l'inspecter |
| **Vérifier les ports** | `docker port <nom/id>` | Liste les mappages de ports réseau du conteneur |

### Nettoyage et Système

| Action | Commande | Description |
| :--- | :--- | :--- |
| **Statistiques** | `docker stats` | Affiche l'utilisation CPU/RAM en temps réel |
| **Inspection globale** | `docker inspect <nom/id>` | Affiche toutes les métadonnées techniques en JSON |
| **Trouver l'IP** | `docker inspect <nom/id> \| grep -i ipaddress` | Extrait rapidement l'adresse IP du conteneur |
| **Nettoyage complet** | `docker system prune -a` | Supprime conteneurs arrêtés, réseaux et images orphelines |

### Docker Compose (Multi-conteneurs)

| Action | Commande | Description |
| :--- | :--- | :--- |
| **Lancer le projet** | `docker compose up -d` | Crée et démarre les services définis en arrière-plan |
| **Arrêter le projet** | `docker compose down` | Arrête et supprime les conteneurs, réseaux et volumes |
| **Voir les états** | `docker compose ps` | Liste les conteneurs du projet actuel |


### Référence

https://docs.docker.com/reference/dockerfile
