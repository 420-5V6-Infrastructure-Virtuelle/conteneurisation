---
title: Idées de TP
weight: 2101
draft: true
---

https://roparst.gricad-pages.univ-grenoble-alpes.fr/cloud-tutorials/docker/


Installation docker
https://blog.stephane-robert.info/docs/conteneurs/moteurs-conteneurs/docker/installation/

## Mini‑exercices pour les étudiants

1. **Démarrer une stack**
   ```bash
   docker compose up -d
   docker compose ps
   ```

2. **Inspecter les logs d’un backend**
   ```bash
   docker compose logs -f backend
   ```

3. **Entrer dans un conteneur DB**
   ```bash
   docker compose exec db bash
   ```

4. **Reconstruire une image**
   ```bash
   docker compose build backend
   docker compose up -d
   ```

5. **Valider un fichier compose**
   ```bash
   docker compose config
   ```

---