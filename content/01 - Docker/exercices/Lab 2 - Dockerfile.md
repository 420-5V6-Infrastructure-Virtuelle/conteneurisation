---
title: "Lab 2 - Dockerfile Linux vs Dockerfile Windows"
weight: 2120
---

#### Dans ce laboratoire, vous allez créer :

1. Un **Dockerfile Linux** basé sur Ubuntu, qui exécute des outils sur Linux.
2. Un **Dockerfile Windows** basé sur Windows Server Core, qui exécute des outils Microsoft.

Ce lab démontre que :

- Un conteneur **Linux** ne peut exécuter que des binaires Linux.
- Un conteneur **Windows** ne peut exécuter que des binaires Windows à moins d'utiliser **WSL**.
- Docker utilise **le noyau du système hôte** pour exécuter les conteneurs.

---

## Dockerfile Linux (Ubuntu)

#### 1. Créez un dossier de travail

```bash
mkdir docker-linux-test
cd docker-linux-test
```

#### 2. Créez un fichier `Dockerfile`

```Dockerfile
FROM ubuntu:22.04

# Mettre à jour et installer des outils Linux
RUN apt update && apt install -y \
    curl \
    iproute2 \
    vim \
    htop

# Commande par défaut
CMD ["bash"]
```

#### 3. Construisez l’image

```bash
docker build -t linux-tools .
```

#### 4. Lancez un conteneur

```bash
docker run -it linux-tools
```

#### 5. Testez des outils Linux dans le conteneur

Dans le conteneur :

```bash
htop
ip a
vim --version
```

#### 6. **Sur le host Ubuntu**, ouvrez un autre terminal et exécutez :
   ```bash
   ps aux | grep htop
   ps aux | grep ip
   ps aux | grep vim
   ```

#### Quel est le résultat ?  (Faites une capture d'écran et collez l'image dans un fichier Word)

</br>

---

## Dockerfile Windows (Windows Server Core)

> Cette partie doit être faite dans votre VM Windows avec Docker Desktop configuré en mode *Windows Containers*.

#### 1. Créez un dossier de travail

```powershell
mkdir docker-windows-test
cd docker-windows-test
```

#### 2. Créez un fichier `Dockerfile`

```Dockerfile
# Image Windows Server Core
FROM mcr.microsoft.com/windows/servercore:ltsc2022

# Installer des outils Windows
RUN powershell -Command \
    Install-WindowsFeature Net-Framework-Features

# Commande par défaut
CMD ["powershell.exe"]
```

#### 3. Construisez l’image

```powershell
docker build -t windows-tools .
```

#### 4. Lancez un conteneur

```powershell
docker run -it windows-tools
```

#### 5. Testez des outils Windows

Dans le conteneur :

```powershell
Get-ComputerInfo
Get-Process
notepad.exe
```

#### 6. Testez un outil Linux (échec attendu)

Toujours dans le conteneur :

```powershell
bash
```

#### 7. **Sur le host Windows**, ouvrez le Gestionnaire des tâches :
   - Ctrl + Shift + Esc
   - Onglet **Processes**
   - Cherchez **notepad.exe**

#### Quel est le résultat du point 5 et 6 ?  (Faites une capture d'écran et collez l'image dans un fichier Word)

</br></br>

---

## Questions à remettre dans votre fichier Word

1. Quelle commande permet de construire une image Docker ?
2. Pourquoi le conteneur Windows ne peut-il pas exécuter `bash` ?
3. Est-ce que Docker émule un OS ?
4. Dans vos mots, quel est le rôle du noyau du host dans l’exécution d’un conteneur ?
5. Quelle est la différence fondamentale entre une vm et un conteneur ?
6. N'oublier pas de mettre les captures d'écran demandées plus haut.

---
