---
title: 02-Les fondamentaux de Kubernetes
draft: false
weight: 3020
---

</br>

### Architecture de K8S

![](../../../images/kubernetes/k8s_archi1.png?width=800px)


#### Cluster
Kubernetes se construit en cluster et fait coopérer des serveurs appelés **noeuds**(nodes).
Un cluster Kubernetes se divise en deux type de noeud distinctes qui ne font pas du tout le même travail :
- Control plan (Master)
- Worker (nœuds de workload)

#### Noeuds Kubernetes

Les nœuds d’un cluster sont les machines (serveurs physiques, machines virtuelles, etc.) qui hébergent les Pods qui sont les composants de la charge de travail des applications. Le **Control plan (master node)** contrôle chaque noeud.

- Pour utiliser Kubernetes, vous utilisez les objets de **l’API Kubernetes** pour décrire l’état souhaité de votre cluster: 
    - Quelles applications ou autres processus que vous souhaitez exécuter, 
    - Quelles images de conteneur elles utilisent, 
    - Le nombre de réplicas, les ressources réseau et disque que vous mettez à disposition, etc .

- Vous créez des objets à l’aide de **l’API Kubernetes**, généralement via l’interface en ligne de commande, **kubectl**. Vous pouvez également utiliser l’API Kubernetes directement pour interagir avec le cluster et définir ou modifier l’état souhaité.

</br>

### Le Control Plane (Master)


Le Control Plane (plan de contrôle) est la partie "intelligente" de Kubernetes. Il prend toutes les décisions du cluster : où déployer vos applications, comment réagir si quelque chose tombe en panne, comment répartir la charge... Mais attention : il ne fait jamais tourner vos conteneurs applicatifs. Son seul travail, c'est d'orchestrer.

Le control plane est responsable du maintien de l’état souhaité pour votre cluster. Lorsque vous interagissez avec Kubernetes, par exemple en utilisatant le cli **kubectl**, vous communiquez avec le control plane de votre cluster via **l'api**.

Il conserve un enregistrement de tous les objets Kubernetes du système et exécute des boucles de contrôle continues pour gérer l’état de ces objets. À tout moment, les boucles de contrôle du control plane répondent aux modifications du cluster et permettent de faire en sorte que l’état réel de tous les objets du système corresponde à l’état souhaité.

Le Control Plane décide et coordonne, il n'exécute aucune charge applicative : vos conteneurs tournent sur les Worker, jamais sur lui

![](../../../images/kubernetes/control-plane-flow.3AGv8Et1_16dNGH.svg)


Le control plane Kubernetes comprend un ensemble de processus en cours d’exécution sur votre cluste. Ces composants ne fonctionnent jamais isolément, ils forment une chaîne de traitement. Voici ce qui se passe quand vous déployez une application :

- `kube-apiserver`: expose l'API pour parler au cluster. C'est ce qui est intérogé lorsque vous utilisez la commandes kubectl
- `etcd`: Est la base de données clé-valeur distribuée, constante et hautement disponible de Kubernetes. Elle sert de « source unique de vérité » au cluster en stockant l'intégralité de son état, sa configuration, ses secrets et la description de toutes ses ressources. **Seul l'API Server y accède directement.**
- `kube-controller-manager`: il surveille en permanence l'état réel du cluster et le compare à l'état souhaité (ce que vous avez défini dans vos fichiers YAML). S'il y a une différence, il agit pour corriger.
- `kube-scheduler`: monitore les resources des différents workers, il cartographie et il décide sur quel worker créé un conteneur(Pods).

### Les noeuds Worker (où tournent vos conteneurs)

Les Worker Nodes (nœuds de travail) sont les machines qui exécutent réellement vos applications. 

C'est là que vos conteneurs tournent, consomment du CPU, de la mémoire, et répondent aux requêtes.

![](../../../images/kubernetes/worker-node-detail.CR0wB8jr_lbXcg.svg)
  
Chaque Worker de votre cluster exécute trois processus :
- `kubelet`, qui communique avec le control plane et controle la création et l'état des pods sur son noeud. Agent qui reçoit les ordres et lance les conteneurs
- `kube-proxy`, un proxy réseau reflétant les services réseau Kubernetes sur chaque nœud. Configure le réseau pour que les Pods communiquent.
- `Container Runtime`, L'environnement d'exécution de conteneurs. Exécute les conteneurs comme docker


### Objets fondamentaux de Kubernetes

- Les **pods** Kubernetes servent à grouper des conteneurs fortement couplés en unités d'application <!-- (microservices ou non) -->
- Les **deployments** sont une abstraction pour **créer ou mettre à jour** (ex : scaler) des groupes de **pods**.
- Enfin, les **services** sont des points d'accès réseau qui permettent aux différents workloads (deployments) de communiquer entre eux et avec l'extérieur.

Au delà de ces trois éléments, l'écosystème d'objets de Kubernetes est vaste et complexe

![](../../../images/kubernetes/k8s_objects_hierarchy.png?width=600px)


### Vue d'ensemble

Vue d'ensemble : nature de chaque concept

| **Concept**     | **Définition en une phrase**                                      | **Commande kubectl associée**          |
|-----------------|-------------------------------------------------------------------|----------------------------------------|
| Cluster         | Ensemble de machines qui exécutent Kubernetes                     | kubectl cluster-info                   |
| Node            | Une machine faisant partie du cluster                             | kubectl get nodes                      |
| Namespace       | Espace logique pour organiser et isoler les ressources            | kubectl get namespaces                 |
| Pod             | Plus petite unité qui exécute un ou plusieurs conteneurs          | kubectl get pods -A                    |
| Deployment      | Contrôleur qui crée, met à jour et maintient les pods             | kubectl get deployments -A             |
| Service         | Point d’accès réseau stable vers un groupe de pods                | kubectl get services -A                |


### Ressources de base
Ces ressources permettent de déployer et exposer vos applications. Maîtrisez-les en premier.

**Pod** </br>
Le Pod est l'unité d'exécution de Kubernetes. Il contient un ou plusieurs conteneurs qui partagent réseau et stockage. Les Pods sont éphémères : ne les créez jamais directement en production.

[→ Apprendre les Pods Kubernetes](https://blog.stephane-robert.info/docs/conteneurs/orchestrateurs/kubernetes/pods/)

**Deployment** </br>
Le Deployment gère le cycle de vie de vos Pods : création, scaling, mises à jour progressives. C'est la ressource que vous utiliserez le plus souvent pour déployer des applications sans état.

[→ Créer et mettre à jour des Deployments Kubernetes](https://blog.stephane-robert.info/docs/conteneurs/orchestrateurs/kubernetes/deployments/)

**ReplicaSet** </br>
Le ReplicaSet maintient un nombre défini de Pods identiques. En pratique, vous ne le créez jamais directement, le Deployment le gère pour vous.

[→ Comprendre les ReplicaSets Kubernetes](https://blog.stephane-robert.info/docs/conteneurs/orchestrateurs/kubernetes/replicasets/)


### Réseau
Ces ressources définissent comment vos applications communiquent entre elles et avec l’extérieur. Maîtriser le réseau est essentiel pour exposer vos services, sécuriser les flux et comprendre le comportement du cluster


**Service** </br>
Un Service expose vos Pods sur le réseau derrière une adresse stable. Comme les Pods sont recréés à tout moment, c'est le Service qui garantit un point d'accès permanent.

[→ Exposer vos applications avec les Services Kubernetes](https://blog.stephane-robert.info/docs/conteneurs/orchestrateurs/kubernetes/services/)


**Ingress & Ingress Controller** </br>
Un Ingress définit des règles d’entrée HTTP/HTTPS pour exposer vos applications vers l’extérieur.
Il nécessite un Ingress Controller (Traefik, Nginx, HAProxy) qui applique ces règles et gère le routage, le TLS, les redirections, etc.

[→ Gérer l’accès HTTP/HTTPS avec les Ingress Kubernetes](https://blog.stephane-robert.info/docs/conteneurs/orchestrateurs/kubernetes/ingress/)

### Stockage
Un Pod est ephémères, donc lorqu'il redémarre et il perd toutes ses données. On a le concept de volumes éphémères pour les données temporaires, et stockage persistant (PV/PVC) pour les données qui doivent survivre aux Pods.

**PersistentVolume (PV)** </br>
Un PersistentVolume représente un volume de stockage réel dans le cluster : disque local, NFS, Ceph, EBS, etc.
Il est provisionné par l’administrateur ou automatiquement via une StorageClass.

**StorageClass** </br>
Une StorageClass définit le type de stockage à utiliser : SSD, HDD, réseau distribué, cloud provider, etc.
Elle permet le provisionnement dynamique : les volumes sont créés automatiquement quand un PVC est demandé.

**Volumes éphémères** </br>
Certains volumes ne sont pas persistants :

[→  Comprendre les volume sur Kubernetes](https://blog.stephane-robert.info/docs/conteneurs/orchestrateurs/kubernetes/storage/)

### Configuration
Ces ressources permettent de configurer vos applications et d'organiser votre cluster.

**ConfigMap** </br>
Un ConfigMap stocke la configuration non sensible de votre application : URLs, paramètres, fichiers de config. Il permet de séparer la configuration du code.

[→ Utiliser les ConfigMaps Kubernetes](https://blog.stephane-robert.info/docs/conteneurs/orchestrateurs/kubernetes/configmaps/)

**Secret** </br>
Un Secret stocke les données sensibles : mots de passe, clés d'API, certificats. Un Secret n'est pas chiffré par défaut, seulement encodé en base64, et l'encodage ne protège rien.

[→ Stocker des données sensibles avec les Secrets Kubernetes](https://blog.stephane-robert.info/docs/conteneurs/orchestrateurs/kubernetes/secrets/)

**Namespace** </br>
Un Namespace isole un groupe de ressources dans le cluster. Il permet de séparer les environnements (dev, staging, prod) ou les équipes.

[→ Comprendre les Namespaces Kubernetes](https://blog.stephane-robert.info/docs/conteneurs/orchestrateurs/kubernetes/namespaces/)

### Ressources avancées
Ces ressources répondent à des besoins spécifiques : applications avec état, agents système et tâches planifiées.

**StatefulSet** </br>
Un StatefulSet gère des applications avec état : bases de données, systèmes distribués. Contrairement au Deployment, il garantit :

Des identités stables (pods numérotés : mysql-0, mysql-1, mysql-2)
Un ordre de démarrage et d'arrêt déterministe
Des volumes persistants liés à chaque pod
[→ Déployer des applications avec état : StatefulSets Kubernetes](https://blog.stephane-robert.info/docs/conteneurs/orchestrateurs/kubernetes/statefulsets/)

**DaemonSet** </br>
Un DaemonSet garantit qu'un Pod tourne sur chaque nœud du cluster. Utilisé pour :

Les agents de monitoring (Prometheus Node Exporter, Datadog)
Les collecteurs de logs (Fluentd, Filebeat)
Les agents réseau (CNI plugins, kube-proxy)
[→ Exécuter un Pod sur chaque nœud : DaemonSets Kubernetes](https://blog.stephane-robert.info/docs/conteneurs/orchestrateurs/kubernetes/daemonsets/)

**Job et CronJob** </br>
Un Job exécute une tâche jusqu'à complétion : migration de base de données, backup, traitement batch. Un CronJob planifie cette exécution de façon récurrente.

[→ Exécuter des tâches avec Jobs et CronJobs Kubernetes](https://blog.stephane-robert.info/docs/conteneurs/orchestrateurs/kubernetes/jobs-cronjobs/)


ref:
https://blog.stephane-robert.info/docs/conteneurs/orchestrateurs/kubernetes/architecture/
https://kubernetes.io/fr/docs/concepts/architecture/