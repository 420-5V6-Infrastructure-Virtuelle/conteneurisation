---
title: "Corrigé 2 - Dockerfile Linux vs Dockerfile Windows"
weight: 2121
draft: true
---


---

## Questions à remettre dans votre fichier Word

1. Quelle commande permet de construire une image Docker ?
2. Pourquoi le conteneur Windows ne peut-il pas exécuter `bash` ?
3. Est-ce que Docker émule un OS ?
4. Dans vos mots, quel est le rôle du noyau du host dans l’exécution d’un conteneur ?
5. Quelle est la différence fondamentale entre une vm et un conteneur ?
6. N'oublier pas de mettre les captures d'écran demandées plus haut.

---

# Corrigé du laboratoire

### **1. Quelle commande permet de construire une image Docker ?**

La commande pour construire une image Docker à partir d’un Dockerfile est :

```bash
docker build -t nom-de-l-image .
```

Le `.` signifie : « utilise le Dockerfile présent dans le dossier courant ».

---

### **2. Pourquoi le conteneur Windows ne peut-il pas exécuter `bash` ?**

Parce que `bash` est un **binaire Linux**, qui dépend du **noyau Linux** pour fonctionner.

Un conteneur Windows utilise le **noyau Windows**, donc il ne peut exécuter que des binaires Windows (ex. `powershell.exe`, `cmd.exe`).  
Les binaires Linux ne sont pas compatibles avec le noyau Windows.

---

### **3. Est-ce que Docker émule un OS ?**

**Non.**

Docker **n’émule pas un système d’exploitation** et **ne virtualise pas un noyau**.  
Docker lance simplement des **processus du host**, isolés dans un environnement contrôlé (namespaces, cgroups, etc.).

C’est pour cela que :

- un conteneur Linux nécessite un **noyau Linux** ;
- un conteneur Windows nécessite un **noyau Windows**.

---

### **4. Dans vos mots, quel est le rôle du noyau du host dans l’exécution d’un conteneur ?**

Le noyau du host est **responsable de l’exécution réelle des processus du conteneur**.

Il fournit :

- la gestion de la mémoire  
- la gestion des processus  
- le système de fichiers  
- le réseau  
- l’isolation (namespaces, cgroups)

Un conteneur n’a **pas son propre noyau** : il utilise celui du host pour tout.

---

### **5. Quelle est la différence fondamentale entre une VM et un conteneur ?**

| Machine virtuelle (VM) | Conteneur |
|------------------------|-----------|
| Possède son **propre noyau** | Utilise le **noyau du host** |
| Virtualise un OS complet | Lance des **processus isolés** |
| Plus lourd (RAM, CPU, disque) | Très léger et rapide |
| Démarre en minutes | Démarre en millisecondes |

 **Différence fondamentale :**  
Une VM virtualise un **système complet**, tandis qu’un conteneur isole des **processus du host**.

---

### **6. N'oubliez pas de mettre les captures d'écran demandées plus haut.**

Les étudiants doivent inclure **deux captures d’écran** :

#### **Capture Linux**
- Le conteneur Linux en train d’exécuter `htop`  
- Le host Ubuntu montrant le processus `htop` via :  
  ```bash
  ps aux | grep htop
  ```

#### **Capture Windows**
- Le conteneur Windows en train d’exécuter `notepad.exe`  
- Le Gestionnaire des tâches du host Windows montrant **notepad.exe** dans la liste des processus

Ces captures démontrent que les processus des conteneurs **roulent réellement sur le host**, et non dans une VM.

---

Si tu veux, je peux aussi te préparer un **corrigé officiel en PDF**, ou un **corrigé version Hugo** pour ton site de cours.