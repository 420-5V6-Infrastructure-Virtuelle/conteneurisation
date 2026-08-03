---
title: "Lab 3 - Docker Compose : Node + MongoDB + React"
weight: 2130
---
</br>

#### Dans ce laboratoire, vous allez :

1. Créer un projet contenant trois services :
   - **backend** : API Node.js (avec un Dockerfile)
   - **database** : MongoDB
   - **frontend** : application React
2. Utiliser `docker-compose.yml` pour orchestrer les trois services.
3. Injecter des données dans MongoDB via un script Node.js.
4. Vérifier que les conteneurs communiquent entre eux.
5. Utiliser VS Code pour vous faciliter la tâche, installer les features docker et YAML

---

## Structure du projet

Vous devez créer la structure suivante :

```
compose-lab/
│
├── backend/
│   ├── Dockerfile
│   ├── package.json
│   ├── index.js
│   └── seed.js   # script d’injection de données
│
├── frontend/
│   └── (fichiers React créés par create-react-app)
│
└── docker-compose.yml
```

---

## Partie 1 — Préparer le backend Node.js

#### 1. Créez le dossier

```bash
mkdir -p compose-lab/backend
cd compose-lab/backend
```

#### 2. Créez `package.json`

```json
{
  "name": "backend",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "seed": "node seed.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "mongodb": "^6.0.0"
  }
}
```

#### 3. Créez `index.js` (API simple)

```js
const express = require("express");
const { MongoClient } = require("mongodb");

const app = express();
const url = "mongodb://database:27017";
const client = new MongoClient(url);

async function main() {
  await client.connect();
  const db = client.db("cours");
  const collection = db.collection("etudiants");

  // Route simple
  app.get("/", (req, res) => {
    res.json({ message: "API Node.js fonctionne !" });
  });

  // Route qui retourne les données insérées par seed.js
  app.get("/etudiants", async (req, res) => {
    const data = await collection.find().toArray();
    res.json(data);
  });

  app.listen(5000, () => console.log("Backend démarré sur le port 5000"));
}

main();
```

#### 4. Créez `seed.js` (injection de données)

```js
const { MongoClient } = require("mongodb");

// Connexion au service "database" défini dans docker-compose
const client = new MongoClient("mongodb://database:27017");

async function run() {
  await client.connect();

  const db = client.db("cours");
  const collection = db.collection("etudiants");

  // Données de test
  await collection.insertMany([
    { nom: "Alice", programme: "Informatique" },
    { nom: "Bob", programme: "Réseaux" },
    { nom: "Charlie", programme: "Développement" }
  ]);

  console.log("Données insérées !");
  await client.close();
}

run();
```

#### 5. Créez le `Dockerfile` du backend

```Dockerfile
# Image Node officielle
FROM node:18

# Dossier de travail dans le conteneur
WORKDIR /app

# Copier package.json et installer les dépendances
COPY package.json .
RUN npm install

# Copier le reste du code
COPY . .

# Exposer le port de l'API
EXPOSE 5000

# Commande de démarrage
CMD ["npm", "start"]

```

---

## Partie 2 — Préparer le frontend React

#### 1. Créez le dossier

```bash
cd ..
mkdir frontend
cd frontend
```

#### 2. Créez l’application React

```bash
npx create-react-app .
```

#### 3. Modifiez `src/App.js`

```js
function App() {
  return (
    <div>
      <h1>Frontend React fonctionne !</h1>
    </div>
  );
}

export default App;
```

---

## Partie 3 — Créer et complété le fichier docker-compose.yml

Dans `compose-lab/`, créez ce fichier componse

compléter les lignes où il est écrit *"Ajouter la ligne :"*

```yaml
version: "3.8"

services:
  # -------------------------
  # Base de données MongoDB
  # -------------------------
  database:
    # Ajouter la ligne : l'image est mongo version 6
    container_name: mongo-db
    # Ajouter la ligne : Expose MongoDB au host. Le port d'entrée et de sortie est 27017
    volumes:
      - mongo-data:/data/db  # Persistance des données

  # -------------------------
  # Backend Node.js (API)
  # Utilise un Dockerfile
  # -------------------------
  backend:
    build: ./backend        # IMPORTANT : on utilise un Dockerfile
    # Ajouter la ligne : Le nom de ce conteneur doit être api-backend
    ports:
      - "5000:5000"
    # Ajouter la ligne : Ce conteneur dépend du conteneur database

  # -------------------------
  # Frontend React
  # Utilise l'image Node
  # -------------------------
  frontend:
    image: node:18
    container_name: react-frontend
    # Ajouter la ligne : le nome du répertoire de travail est "/app"
    volumes:
      - ./frontend:/app     # Monte le code React
    ports:
      - "3000:3000"
    command: ["npm", "start"]  # Lance le serveur de dev React

# -------------------------
# Volume persistant MongoDB
# -------------------------
volumes:
  mongo-data:

```

---

## Partie 4 — Démarrer l’environnement

```bash
docker compose up -d --build
```

---

## Partie 5 — Injecter des données dans MongoDB

```bash
docker compose exec backend npm run seed
```

Vérifiez dans MongoDB :

```bash
docker compose exec database mongosh
> use cours
> db.etudiants.find()
```

---

## 🧠 Questions à remettre dans votre fichier Word

1. Quelle commande permet de démarrer un environnement Docker Compose ?
2. Quelle commande permet d’exécuter un script dans un conteneur ?
3. Expliquez comment les services communiquent entre eux dans Docker Compose.
4. Quelle est la différence entre `image:` et `build:` dans docker-compose ?
5. Montrez une capture d’écran de vos trois conteneurs en cours d’exécution.
6. Montrez une capture d’écran de vos données MongoDB insérées via `seed.js`.

---

## ❓ Est-ce qu’il existe des images avec des données de test ?

Oui, mais **elles ne sont pas officielles**.

- Il existe des images MongoDB pré-remplies sur Docker Hub, mais elles sont créées par la communauté.
- Elles ne sont **pas recommandées** pour un cours, car :
  - elles ne sont pas maintenues ;
  - elles peuvent contenir des données douteuses ;
  - elles ne sont pas pédagogiques.

👉 **La meilleure pratique est de fournir ton propre script `seed.js`**, comme dans ce lab.

