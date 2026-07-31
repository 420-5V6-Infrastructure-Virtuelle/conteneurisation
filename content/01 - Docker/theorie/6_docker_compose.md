---
title: Docker Compose
weight: 2080
---

Docker Compose est un outil qui permet de définir et lancer plusieurs conteneurs Docker comme une seule application. Il utilise un fichier YAML pour décrire les services, réseaux et volumes, simplifiant l’orchestration. C’est une approche déclarative typique de la philosophie IaC (infrastructure as code).

En d'autres mots, au lieu d'exécuter plusieurs lignes de commande `docker run` avec des options complexes, vous décrivez votre architecture dans un fichier YAML et lancez tout avec Docker compose up.

Docker Compose automatise le démarrage, l’arrêt et la coordination des conteneurs, rendant le développement multiservice plus rapide, reproductible et facile à gérer.

Malgré la domination de Kubernetes dans les déploiements à grande échelle, Docker Compose demeure l’outil de prédilection pour du déploiement local.

Cela est utile pour faire des tests, apprendre comment monter des environnements, monter des architectures 3-tiers ou reproduire un environnement de prod sur votre poste.

**On peut aussi appeler un Dockerfile à partir d'un Docker Compose**


<img src="../../../images/docker/3-tiers-Compose.jpg" width="75%">


### Installation de Docker compose (si nécessaire)

```bash
# Vérifier s'il y est installé
docker --version
docker compose version

# pour l'installer

# Ubuntu/Debian
sudo apt update
sudo apt install docker-compose-plugin

# Vérification
docker compose version
```


#### concepts fondamentaux

Avant d'écrire votre premier fichier, comprenez ces trois concepts fondamentaux.

##### Service

-   Un service dans Docker Compose représente un conteneur logique. 
-   Il décrit l’image à utiliser, les ports, les variables d’environnement, les volumes et les dépendances. 
-   Chaque service correspond à une partie autonome de l’application, orchestrée par Compose.

Chaque service peut définir :
-   L'**image** Docker à utiliser
-   Les **ports** à exposer vers l'extérieur ou entre services
-   Les **variables d'environnement** pour configurer l'application
-   Les **volumes** à monter pour persister ou partager des données
-   Les **dépendances** vers d'autres services (qui doivent démarrer en premier)
-   Les **ressources** (limites CPU, mémoire) et les politiques de redémarrage

**Exemple d'utilisation** : Dans une architecture 3‑tiers, nous définissons trois services dans notre fichier Docker Compose :
-   Frontend pour l’interface utilisateur (React)
-   Backend pour la logique métier (Node.js)
-   Base de données pour la persistance (MongoDB)

##### [Network](../4_reseaux/#comment-lier-un-type-de-réseau-à-un-conteneur)
-   Un réseau Docker Compose permet aux services de communiquer automatiquement entre eux.
-   Comme un réseau local virtuel, les conteneurs connectés au même réseau peuvent se voir, ceux sur des réseaux différents sont isolés. 
-   Chaque conteneur peut communiquer en utilisant le nom du service, sans configuration supplémentaire (sans faire de configuration IP).

##### [Volume](../4_volumes/#les-volumes-docker-via-la-sous-commande-volume)
-   Fournis un stockage persistant, partagé ou monté dans un service
-   Souvent utilisé dans la gestion des bases de données ou fichiers applicatifs.
-   Un volume résout le problème de la nature éphémère du conteneur en stockant les données en dehors du conteneur

**Voici un tableau avec les éléments le plus courant d'un docker-compose**

| Concept | Description | Exemple |
|--------|-------------|---------|
| **Service** | Un conteneur logique défini dans le fichier Compose. | `services:`<br><dd>`frontend:` |
| **Image** | Source du conteneur : image existante ou construite via `build:`. | `image: mongo:latest`<br>`build: ./frontend` |
| **Conteneur** | Instance en exécution d’une image, gérée par Compose. | `docker compose up` |
| **Network** | Réseau interne reliant les services automatiquement. | Communication `backend → database` |
| **Volume** | Stockage persistant partagé ou monté dans un conteneur. | `volumes:`<br>`db_data:/var/lib/mysql` |
| **Environment Variables** | Configuration injectée dans les conteneurs. | `environment:`<br><dd>`- MONGO_URI=...` |
| **Dependencies** | Ordre de démarrage entre services. | `depends_on:`<br><dd>`- database` |
| **Compose File** | Fichier YAML décrivant l’architecture multiconteneurs. | `docker-compose.yml` |
| **Commandes Compose** | Gestion du cycle de vie des conteneurs. | `docker compose up`, `down`, `logs` |


#### Exemple complet d'un Docker compose

```yaml
# ==============================================================================
# EXEMPLE : ARCHITECTURE MULTI-CONTENEURS AVEC DOCKER COMPOSE
# ==============================================================================

services:

  # ----------------------------------------------------------------------------
  # 1. LE SERVICE DE BASE DE DONNÉES (Couche persistance - MySQL)
  # ----------------------------------------------------------------------------
  db:
    # Toujours cibler une version spécifique (ici une version LTS) en 
    # développement et production. Éviter ':latest' qui peut briser le projet du jour au lendemain.
    image: mysql:8.4
    
    # Nommer le conteneur évite que Docker génère un nom aléatoire.
    # Utile pour vos commandes CLI : 'docker logs cegep-mysql-db' ou 'docker exec -it ...'
    container_name: cegep-mysql-db  
    
    # Résilience. 'always' force le conteneur à redémarrer s'il plante ou si 
    # l'ordinateur/serveur redémarre.
    restart: always                  
    
    environment:
      # Sécurité et flexibilité. La syntaxe ${VAR:-defaut} tente de lire une 
      # variable dans un fichier '.env' local. Si elle n'existe pas, elle prend la valeur après le ':-'.
      # C'est la bonne pratique pour ne JAMAIS pousser de vrais mots de passe sur GitHub.
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD:-changeme_root}
      MYSQL_DATABASE: ${MYSQL_DB_NAME:-app_database}
      MYSQL_USER: ${MYSQL_USER:-app_user}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD:-changeme_user}
      
    volumes:
      # Persistance des données. Un conteneur est éphémère (si on le supprime, on perd tout).
      # On mappe un "Volume nommé" (géré par Docker) vers le dossier interne de MySQL.
      - db_data:/var/lib/mysql
      
      # CONCEPT (Astuce de prof) : Si vous décommentez la ligne sous ce commentaire, tout les scripts 
      # .sql placés dans votre dossier local './scripts_init' seront exécutés AUTOMATIQUEMENT 
      # au tout premier démarrage de la base de données. Très utile pour injecter un schéma et générer une bd !
      # - ./scripts_init:/docker-entrypoint-initdb.d
      
    networks:
      # Sécurité par isolation. La base de données est placée UNIQUEMENT dans 
      # le réseau arrière-plan (backend). Elle est invisible depuis l'extérieur ou depuis le serveur Web.
      - backend-network              
      
    healthcheck:
      # Validation d'état (Crucial !). Par défaut, Docker considère qu'un conteneur 
      # est prêt dès que son processus démarre. Mais MySQL prend souvent 10-15 secondes pour s'initialiser.
      # Ce test exécute un 'ping' interne pour confirmer que MySQL est RÉELLEMENT prêt à répondre.
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "root", "-p$MYSQL_ROOT_PASSWORD"]
      interval: 10s   # Temps entre chaque vérification
      timeout: 5s     # Temps max accordé à la commande pour répondre
      retries: 5      # Nombre d'échecs consécutifs avant de déclarer le conteneur "unhealthy"

  # ----------------------------------------------------------------------------
  # 2. L'APPLICATION NODE.JS (Couche Logique / API Backend)
  # ----------------------------------------------------------------------------
  app:
    # Les images '-alpine' sont basées sur une distribution Linux ultra-légère (~5 Mo).
    # Cela permet de télécharger et déployer les conteneurs beaucoup plus rapidement au laboratoire.
    image: node:20-alpine            
    container_name: cegep-node-app
    restart: unless-stopped          # Redémarre sauf si l'utilisateur l'a arrêté manuellement (ex: docker compose down)
    
    # Définit le dossier de travail PAR DÉFAUT à l'intérieur du conteneur.
    working_dir: /usr/src/app
    
    volumes:
      # Bind Mount (Montage lié) pour le développement. 
      # On lie le dossier './backend' de votre ordinateur au dossier '/usr/src/app' du conteneur.
      # Ainsi, dès qu'un étudiant modifie son code dans VS Code, le conteneur voit le changement 
      # instantanément (permet le Live Reload / Hot Reload avec nodemon).
      - ./backend:/usr/src/app
      
    environment:
      NODE_ENV: development
      # Le DNS Interne de Docker. Pour se connecter à la base de données, on n'utilise PAS 
      # une adresse IP (ex: 172.18.0.2) car elle change tout le temps. On utilise simplement le NOM 
      # du service Docker, soit 'db'. Docker résout le nom automatiquement !
      DB_HOST: db                    
      DB_USER: ${MYSQL_USER:-app_user}
      DB_PASS: ${MYSQL_PASSWORD:-changeme_user}
      DB_NAME: ${MYSQL_DB_NAME:-app_database}
      
    ports:
      # Mappage de ports (HÔTE:CONTENEUR). Le port 3000 de votre machine physique 
      # est redirigé vers le port 3000 interne du conteneur. Utile pour tester l'API directement 
      # avec Postman ou l'extension Thunder Client.
      - "3000:3000"                  
      
    # Remplace l'instruction CMD du Dockerfile d'origine. Ici, on lance le script de 
    # développement défini dans le fichier package.json de l'étudiant.
    command: npm run dev             
    
    networks:
      # Le pont (Bridge). Ce conteneur a les pieds dans DEUX réseaux.
      # Il peut parler à la base de données (via backend-network) ET recevoir les requêtes de Nginx 
      # (via frontend-network). C'est le seul intermédiaire.
      - backend-network              
      - frontend-network             
      
    depends_on:
      db:
        # Ordonnancement intelligent. Ne démarre pas Node.js tant que le service 'db' 
        # n'a pas passé son test 'healthcheck' avec succès. Fini les erreurs "Connection refused" !
        condition: service_healthy   

  # ----------------------------------------------------------------------------
  # 3. LE SERVEUR WEB / REVERSE PROXY (Couche Présentation - Nginx)
  # ----------------------------------------------------------------------------
  web:
    image: nginx:alpine
    container_name: cegep-nginx-web
    restart: always
    
    ports:
      # Exposition publique. On mappe le port standard HTTP (80) de la machine hôte. 
      # C'est la seule porte d'entrée officielle pour l'utilisateur final (ex: http://localhost).
      - "80:80"                      
      
    volumes:
      # Injection de configuration et sécurité. On remplace la configuration par défaut 
      # de Nginx par la nôtre. L'ajout du drapeau ':ro' (Read-Only) est une bonne pratique de sécurité : 
      # le conteneur peut lire le fichier, mais ne pourra jamais le modifier ou le corrompre.
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf:ro  
      
      # CONCEPT OPTIONNEL : Si les étudiants ont un projet Frontend (React, Vue, HTML statique), 
      # Nginx peut servir les fichiers compilés directement sans passer par Node.js, ce qui est beaucoup plus performant.
      # - ./frontend/dist:/usr/share/nginx/html:ro
      
    networks:
      # Sécurité (Principe du moindre privilège). Nginx reçoit les requêtes du Web, il est 
      # le plus vulnérable. En l'isolant dans le 'frontend-network', s'il se fait pirater, l'attaquant 
      # n'a aucun chemin réseau pour atteindre directement la base de données.
      - frontend-network             
      
    depends_on:
      # Nginx a besoin que l'application Node tourne pour pouvoir lui relayer les requêtes API.
      - app
      # Avec Compose v3+ -> depends_on: condition: service_healthy

# ==============================================================================
# DÉCLARATION DES INFRASTRUCTURES PARTAGÉES (Réseaux et Volumes)
# ==============================================================================

networks:
  # Par défaut, Docker utilise le pilote (driver) 'bridge' (un commutateur/switch virtuel) 
  # pour créer des sous-réseaux isolés sur la machine hôte.
  frontend-network:
    driver: bridge
  backend-network:
    driver: bridge

volumes:
  # Les volumes nommés sont persistants et gérés par le moteur Docker. Même si l'étudiant 
  # fait un 'docker compose down', les données de la base de données restent en sécurité sur son volume.
  db_data:
    # Assigner un nom explicite permet à l'étudiant de repérer immédiatement son volume 
    # lorsqu'il exécute la commande 'docker volume ls'.
    name: cegep_mysql_data
```

#### Vérifier que tous les conteneurs sont démarrés

```bash
docker compose ps

NAME              IMAGE           COMMAND                  SERVICE   STATUS              PORTS
cegep-mysql-db    mysql:8.4       "docker-entrypoint.s…"   db        running (healthy)   3306/tcp, 33060/tcp
cegep-node-app    node:20-alpine  "docker-entrypoint.s…"   app       running             0.0.0.0:3000->3000/tcp
cegep-nginx-web   nginx:alpine    "/docker-entrypoint.…"   web       running             0.0.0.0:80->80/tcp

# reste qu'à tester avec le navigateur http://localhost

```

Le "langage" de Docker Compose : [la documentation du langage (DSL) des compose-files](https://docs.docker.com/compose/compose-file/) est essentielle.

**Rappel**

Faire un exemple complet et le mettre sur git hub un peu comme https://tech-insider.org/fr/docker-compose-tutoriel-stack-production-13-etapes-2026/


### Commandes Docker Compose (CLI)

#### 1. Démarrer et arrêter une stack

`docker compose up`
- Lance tous les services définis dans le fichier.  

Options utiles :
- `-d` : mode détaché (lance les conteneurs en arrière-plan)  
- `--build` : force la reconstruction des images  
- `--pull` : force le téléchargement des images  

**Exemples :**
```bash
docker compose up
docker compose up -d
docker compose up --build
```

---

`docker compose down`
Arrête et supprime les conteneurs.  
Ne supprime pas les volumes.

**Options :**
- `--volumes` : supprime aussi les volumes nommés  
- `--remove-orphans` : supprime les conteneurs non définis dans le compose  

**Exemples :**
```bash
docker compose down
docker compose down --volumes
```

---

#### 2. Gestion des conteneurs

`docker compose ps`
Affiche les conteneurs de la stack, leurs ports et leur état.

```bash
docker compose ps
```

---

`docker compose restart`
Redémarre un service ou toute la stack.

```bash
docker compose restart
docker compose restart backend
```

---

 `docker compose stop` / `docker compose start`
Arrête ou démarre les conteneurs sans les supprimer.

```bash
docker compose stop
docker compose start
```

---

#### 3. Logs et débogage

`docker compose logs`
Affiche les logs de tous les services.

**Options :**
- `-f` : suivi en temps réel  
- `service` : logs d’un service spécifique  

```bash
docker compose logs
docker compose logs -f backend
```

---

`docker compose exec`
Exécute une commande dans un conteneur en cours d’exécution.

```bash
docker compose exec backend bash
docker compose exec db psql -U postgres
```

---

`docker compose run`
Lance un conteneur temporaire basé sur un service.

```bash
docker compose run --rm backend ls /app
```

---

#### 4. Construction et images

`docker compose build`
Construit les images définies avec `build:`.

```bash
docker compose build
docker compose build backend
```

---

`docker compose pull`
Télécharge les images depuis un registre.

```bash
docker compose pull
```

---

#### 5. Inspection de la configuration

`docker compose config`
Valide le fichier YAML et affiche la configuration finale.

```bash
docker compose config
```

---

#### 6. Commandes avancées (optionnelles)

`docker compose top`
Affiche les processus actifs dans chaque conteneur.

`docker compose cp`
Copie des fichiers entre l’hôte et un conteneur.

`docker compose events`
Affiche les événements Docker en temps réel.

---



