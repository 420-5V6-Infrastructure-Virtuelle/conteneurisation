---
title: "Cours 1 - Installer et explorer Docker sur Linux et Windows"
weight: 2110
---

### Installer Docker Engine sur Ubuntu

- Accédez à votre serveur Proxmox et réutilisez ou créez une vm Ubuntu **mise à jour**.

- Pour installer Docker :
   - Suivez la [documentation officielle pour installer Docker sur Ubuntu](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)
   - Faites les points 1 et 2 de la section **(*Install using the apt repository*)**

- Lancez `sudo docker run hello-world`. Que s'est-il passé ?
   - Copiez l’image du résultat dans votre fichier Word.

</br>

> **Il manque les droits pour exécuter Docker sans utiliser `sudo` à chaque fois.**
> - Le daemon tourne toujours en `root`.
> - Un utilisateur ne peut accéder au client que s’il est membre du groupe `docker`.
> - Ajoutez-le au groupe avec la commande : `sudo usermod -aG docker $USER`
> - Pour actualiser la liste des groupes auxquels appartient l’utilisateur, redémarrez la vm avec `sudo reboot`, puis reconnectez-vous avec Guacamole pour que la modification prenne effet.

</br>

#### Autocomplétion

- Pour vous faciliter la vie, ajoutez le plugin d’autocomplétion pour Docker et Docker Compose à `bash` en copiant les commandes suivantes :

```bash
sudo apt update
sudo apt install bash-completion curl
sudo curl -L https://raw.githubusercontent.com/docker/compose/1.24.1/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose
```

**Important:** Vous pouvez désormais appuyer sur la touche <TAB> pour utiliser l'autocomplétion quand vous écrivez des commandes Docker

---

#### Vérifier l'installation

- Les commandes de base pour connaître l'état de Docker sont :

```bash
docker info  # affiche de nombreuses informations sur l'engine Docker
docker ps    # affiche les conteneurs en cours d'exécution
docker ps -a # affiche également les conteneurs arrêtés
```


**Sauvegarder cette vm comme modèle au nom de "docker-linux"**

---

### Installer Docker Desktop sur Windows

- Accédez à votre serveur Proxmox et réutilisez ou créez un vm Windows **mise à jour**.

<!-- - Vérifiez l'installation de Docker en lançant `sudo docker info`. -->

- Pour installer Docker Desktop (avec l'utilisation de WSL)
   -  Suivez la [documentation officielle pour installer Docker sur Windows](https://docs.docker.com/desktop/setup/install/windows-install/)
   -  Faites du début et arrêtez-vous à la section **(*Advanced system configuration and installation options*)**
   -  Vérifiez l'installation et que toutes les [permissions Windows requises soient configurées](https://docs.docker.com/desktop/setup/install/windows-permission-requirements/)