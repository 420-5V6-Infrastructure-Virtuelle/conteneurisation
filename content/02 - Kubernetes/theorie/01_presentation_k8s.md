---
title: 01-Présentation de Kubernetes
draft: false
weight: 3010
---

</br>

### Un peu d'histoire

Tout commence dans les années 1980 avec la commande Unix chroot, qui permettait de modifier le répertoire racine d’un processus. Ce n’était pas encore de la conteneurisation à proprement parler, mais déjà un premier pas vers l’isolation de processus.

Dans les années 2000, FreeBSD introduit les jails, des environnements isolés capables de faire tourner des applications en toute sécurité. De son côté, Solaris propose les zones, qui poussent encore plus loin la séparation des ressources système.

Tout bascule en 2013 avec Docker, qui démocratise la conteneurisation grâce à une interface simple et un format standardisé. Très vite, le besoin de gérer des dizaines, voire des centaines de conteneurs en production se fait sentir. C’est là qu’intervient l’orchestration.

En 2014, Google ouvre Kubernetes (ou K8s) en open source. C’est une révolution : Kubernetes devient rapidement la solution de référence pour gérer des clusters de conteneurs. Ce projet s’appuie sur l’expérience de Google avec son système interne Borg, utilisé en production depuis des années.Google l'utilise pour déployer et gérer des milliers d’applications dans leurs data centers, assurant ainsi une haute disponibilité et une gestion efficace des ressources.

En 2015, Kubernetes a été transféré sous l’égide de la [Cloud Native Computing Foundation](https://fr.wikipedia.org/wiki/Cloud_Native_Computing_Foundation)

Depuis son lancement, Kubernetes a évolué rapidement grâce à une communauté active et une forte adoption par des entreprises de toutes tailles.

Les contributions de divers acteurs de l’industrie comme IBM, Microsoft, Red Hat et d’autres ont enrichi le projet en ajoutant des fonctionnalités avancées et en améliorant sa stabilité et sa performance.

</br>

### Trois transformations profondes actuelles de l'informatique

Kubernetes se trouve au coeur de trois transformations profondes techniques, humaines et économiques de l'informatique:

- Le cloud
  - Permet de mettre en place des architectures infonuagique
  - Offre la redondance et la résiliance nécessaire pour des solutions cloud
- La conteneurisation logicielle
  - La solution la plus utiliser dans des architectures complexes
- Le mouvement DevOps
  - Permet de manière efficace la segmentation des différents environnements (DEV, QA, Staging, PréPROD et PROD)
  - Facilite la mise en place de CICD

Il est un des projets qui symbolise et supporte techniquement ces transformations. D'où son omniprésence dans les monde informatiques actuellement.

</br>

### Concrètement : Architecture de Kubernetes

![](../../../images/kubernetes/k8s_archi1.png?width=800px)

- Kubernetes rassemble en un cluster et fait coopérer un groupe de serveurs appelés **noeuds**(nodes).

- Kubernetes a une architecture **Master/workers** (cf. cours 2) composée d'un **control plane** et de nœuds de calculs (**workers**).

- Cette architecture permet essentiellement de rassembler les machines en un **cluster unique** sur lequel on peut faire tourner des **"charges de calcul" (workloads)** très diverses.

- Sur un tel cluster le déploiement d'un workload prend la forme de **ressources (objets k8s)** qu'on **décrit sous forme de code** et qu'on crée ensuite effectivement via l'API Kubernetes.

- Pour uniformiser les déploiement logiciel Kubernetes est basé sur le standard des **conteneurs** (défini aujourd'hui sous le nom **Container Runtime Interface**, Docker est l'implémentation la plus connue).

- Plutôt que de déployer directement des conteneurs, Kubernetes crée des **aggrégats de un ou plusieurs conteneurs** appelés des **Pods**. Les pods sont donc l'unité de base de Kubernetes.

</br>

### Objets fondamentaux de Kubernetes

- Les **pods** Kubernetes servent à grouper des conteneurs fortement couplés en unités d'application <!-- (microservices ou non) -->
- Les **deployments** sont une abstraction pour **créer ou mettre à jour** (ex : scaler) des groupes de **pods**.
- Enfin, les **services** sont des points d'accès réseau qui permettent aux différents workloads (deployments) de communiquer entre eux et avec l'extérieur.

Au delà de ces trois éléments, l'écosystème d'objets de Kubernetes est vaste et complexe

![](../../../images/kubernetes/k8s_objects_hierarchy.png?width=600px)

</br>

#### Kubernetes entre Cloud et auto-hébergement

Un des intérêts principaux de Kubernetes est de fournir un modèle de Plateform as a Service (PaaS) suffisamment versatile qui permet l'interopérabilité entre des fournisseurs de clouds différents et des solutions auto-hébergées (on premise).

Cependant cette interopérabilité n'est pas automatique (pour les cas complexes) car Kubernetes permet beaucoup de variations. Concrètement il existe des variations entre les installations possibles de Kubernetes

</br>

---

### Distributions et "flavours" de Kubernetes

Kubernetes est avant tout un ensemble de standards qui peuvent avoir des implémentations concurrentes. Il existe beaucoup de variétés (**flavours**) de Kubernetes, implémentant concrètement les solutions techniques derrière tout ce que Kubernetes ne fait que définir : solutions réseau, stockage (distribué ou non), loadbalancing, service de reverse proxy (Ingress), autoscaling de cluster (ajout de nouvelles VM au cluster automatiquement), monitoring…

Les **services gérés des Cloud Providers (Managed Kubernetes)** dominent très largement le marché (près de 80 % de l'ensemble des clusters en production), suivis des **distributions légères et spécialisées**. 

Une forte tendance à la hausse concerne les déploiements destinés à l'**Edge computing**, aux plateformes de dev (*Platform Engineering*) et aux **workloads IA/Machine Learning**.
</br>
</br>
</br>




#### Voici les distributions les plus populaires & en forte hausse

</br>

#### A. Les géants du Cloud (Public Managed K8s)
Elles représentent la majorité du marché global et continuent de croître à mesure que les entreprises migrent leurs applications vers le cloud.

* **Amazon EKS (AWS) :** Le leader incontesté en termes de parts de marché (environ 40-42 %).
* **Google GKE (Google Cloud) :** Le pionnier technique. Son option **GKE Autopilot** (gestion 100 % automatisée des nœuds et des coûts) connaît une hausse d'adoption massive.
* **Azure AKS (Microsoft) :** Très forte progression en entreprise grâce à son intégration native avec l'écosystème Microsoft (Entra ID, Azure DevOps).

#### B. Les distributions légères & Edge (Edge, IoT, CI/CD)
C'est la catégorie dont la **tendance est la plus fortement à la hausse**, tirée par l'Edge Computing et le besoin d'outils plus simples.

* **K3s (SUSE/Rancher) :** La distribution ultra-légère de référence (binaire < 100 Mo). Elle explose sur le marché grâce aux cas d'usage IoT, Edge et dev rapide. Contrairement à K8S, K3S remplace l'exigeante base etcd par SQLite par défaut et intègre nativement son runtime (containerd), son réseau (Flannel) et son contrôleur d'Ingress (Traefik), réduisant ainsi l'empreinte RAM de 2 Go à seulement 512 Mo. **C'est cette version que nous allons utiliser pour les lab.**
* **Talos Linux :** **Très forte hausse (Tendance "Immutable Infrastructure").** Talos n'est pas qu'une distribution K8s, c'est un OS minimaliste sécurisé sans shell ni SSH, administré à 100 % par API. C'est le favori montant des équipes Ops pour la sécurité de leurs nœuds Bare-Metal ou Cloud.
* **k0s (Mirantis) :** Distribution distribuée sous forme de binaire unique, en nette progression pour sa simplicité d'installation dans n'importe quel environnement (Edge à On-Premise).

#### C. Les plateformes Enterprise / Hybrides
* **Red Hat OpenShift :** Reste le standard absolu sur le marché des entreprises traditionnelles (banques, assurances, secteur public) nécessitant une plateforme sécurisée clé en main ("opinionated") sur site ou en Cloud Hybride.
* **RKE2 (SUSE / Rancher) :** En hausse constante dans les secteurs nécessitant un haut niveau de sécurité (conformité FIPS, gouvernements) et la gestion centralisée multi-cluster.

#### D. Développement local & Tests
* **Kind (Kubernetes in Docker) :** En constante augmentation au détriment de Minikube pour les pipelines de CI/CD automatisés, car il fait tourner les nœuds K8s directement dans des conteneurs Docker.

---

#### Synthèse des tendances du marché

| Distribution | Catégorie | Tendance du marché | Cas d'usage principal |
| :--- | :--- | :--- | :--- |
| **Google GKE (Autopilot)** | Cloud Managed | ⬆️ Forte Hausse | Production Cloud & Workloads IA |
| **K3s** | Minimaliste / Edge | ⬆️ Exponentielle | Edge Computing, IoT, CI/CD |
| **Talos Linux** | OS Sécurisé / Immutable | ⬆️ Très Forte Hausse | Infra Bare-Metal & Zero-Trust Security |
| **Amazon EKS** | Cloud Managed | ➡️ Dominant / Stable | Production Cloud générale |
| **Red Hat OpenShift** | Enterprise Platform | ➡️ Stable / Solide | Cloud Hybride & Secteurs régulés |