---
title: Réseaux
weight: 2050
---


Docker propose 6 drivers réseau natifs, chacun adapté à un cas d'usage spécifique. Le choix du bon réseau impacte directement la sécurité, les performances et la facilité de maintenance de vos applications conteneurisées.

Ces modes réseau permettant de contrôler l’isolation, la performance, et la manière dont les conteneurs communiquent entre eux ou avec l’extérieur.
Chaque type répond à un besoin précis : développement, production distribuée, intégration dans un réseau physique, sécurité, etc.

![](../../../images/docker/docker-network-types.C7umlaTg_23LqvG.svg)


| Type     | Isolation       | Performance     | Multihôtes | Cas d’usage principal                                   |
|----------|------------------|------------------|-------------|----------------------------------------------------------|
| Bridge   | ✓ Bonne          | Bonne            | ✗ Non       | Développement, apps multiconteneurs                    |
| Host     | ✗ Aucune         | ⚡ Maximale       | ✗ Non       | Performance critique, monitoring                         |
| Overlay  | ✓ Bonne          | Bonne            | ✓ Oui       | Production distribuée, Swarm                             |
| Macvlan  | ✓ Excellente     | ⚡ Très bonne     | ✗ Non       | Applications legacy, intégration réseau physique         |
| Ipvlan   | ✓ Excellente     | ⚡ Très bonne     | ✗ Non       | Environnements avec restrictions MAC                     |
| None     | 🔒 Totale        | N/A              | ✗ Non       | Sécurité maximale, jobs batch                            |


#### 1. Bridge - le réseau par défaut
Le mode bridge est utilisé automatiquement lorsque tu crées un conteneur sans préciser de réseau.

##### Fonctionnement
  - Docker crée un réseau virtuel interne (ex. bridge0).
  - Les conteneurs reçoivent une IP interne (ex. 172.17.0.x).
  - La communication entre conteneurs passe par ce réseau.
  - L’accès Internet se fait via NAT.

##### Cas d’usage
  - Développement local.
  - Applications multiconteneurs simples.
  - Scénarios où tu veux isoler les conteneurs du réseau de l’hôte.

##### Analogie
  - Un réseau bridge, c'est comme un switch réseau virtuel installé sur votre machine. Chaque conteneur connecté reçoit une adresse IP privée et peut parler aux autres conteneurs du même commutateur.

> ##### ⚠️ Attention - Réseau *bridge* par défaut
> Le bridge par défaut (docker0) ne permet pas la résolution DNS par nom parce qu'il n'a pas de DNS interne. Vous devez utiliser les adresses IP, ce qui rend votre configuration fragile. 
> Les réseaux bridge personnalisés créés avec docker network create ont un DNS interne permettant la résolution par nom.
>
> Créez toujours un bridge personnalisé pour vos applications


#### 2. Host - pas d’isolation réseau
Le conteneur partage directement la pile réseau de l’hôte.

##### Fonctionnement
  - Pas d’IP propre au conteneur.
  - Le conteneur utilise directement les ports de l’hôte (sans NAT).
  - Performance maximale, aucune couche réseau Docker.
  - Accès réseau identique à celui de l’hôte.

##### Cas d’usage
  - Monitoring (Prometheus, node-exporter).
  - Applications nécessitant un accès réseau ultrarapide.
  - Scénarios où l’isolation réseau n’est pas nécessaire.

##### Analogie
  - Si le bridge est un switch virtuel, le mode host c'est comme brancher directement votre conteneur sur la carte réseau de la machine. Aucun intermédiaire, aucune traduction d'adresses.

#### 3. Overlay - réseau multihôte
Permets à des conteneurs situés sur plusieurs serveurs de communiquer comme s’ils étaient sur le même réseau.

##### Fonctionnement
  - Utilisé par Docker Swarm ou Kubernetes (via CNI - Container Network Interface).
  - Encapsulation réseau via VXLAN.
  - Réseau virtuel distribué entre plusieurs nœuds.
  - Communication interserveur transparente.

##### Cas d’usage
  - Microservices distribués.
  - Clusters Swarm.
  - Environnements de production multiserveur.

##### Analogie
  - Un overlay, c'est comme un VPN privé entre vos serveurs Docker. Les conteneurs pensent être sur le même réseau local, même s'ils sont physiquement sur des machines différentes.

#### 4. Macvlan - conteneurs avec une IP du réseau physique
Le conteneur apparaît comme une machine physique sur le réseau.

##### Fonctionnement
  - Le conteneur reçoit une vraie adresse IP du LAN.
  - Le conteneur est visible comme un hôte distinct par les routeurs/switches.
  - Permets d’éviter le NAT.
  - Idéal pour les applications nécessitant une présence directe sur le réseau.

##### Cas d’usage
  - Applications legacy.
  - Services réseau (DHCP, DNS, VPN).
  - Intégration directe dans un réseau d’entreprise.


##### Analogie
  - Avec macvlan, votre conteneur obtient sa propre carte réseau virtuelle. Du point de vue du réseau, c'est comme si vous aviez branché un nouvel ordinateur sur le switch.

#### 5. Ipvlan - variante moderne de Macvlan
Similaire à Macvlan, mais gère différemment les adresses MAC.

##### Fonctionnement
  - Le conteneur reçoit une IP du réseau physique.
  - Une seule adresse MAC est utilisée pour plusieurs conteneurs.
  - Réduis les problèmes dans les environnements où les adresses MAC sont limitées.
  - Fonctionne mieux dans les réseaux très contrôlés.

##### Cas d’usage
  - Datacenters avec restrictions sur les adresses MAC.
  - Réseaux d’entreprise très stricts (Cisco, Juniper).
  - Scénarios nécessitant une intégration réseau directe sans surcharge MAC.


#### Comment lier un type de réseau à un conteneur

##### Avec docker run

```bash
docker run --network bridge nginx
docker run --network host nginx
docker run --network none nginx
docker run --network my-macvlan nginx
```

##### Réseau personnalisé

Pour overlay, macvlan, ipvlan, tu dois créer le réseau avant :
```bash
docker network create -d macvlan my-macvlan -o parent=eth0
docker run --network my-macvlan nginx
```

##### Réseau bridge - Si on ne veut pas utiliser le bridge par défaut (Docker0) 
Le bridge par défaut n'est pas configurable, pas sécurisé et ne met pas à jour proprement les tables DNS.
C'est pour une de ces raisons qu'on n'utilise jamais le bridge par défaut en production
```bash
# Création d'un bridge personnalisé
docker network create --driver bridge --subnet 10.10.0.0/24 --gateway 10.10.0.1 app_network

# Lancez la base de données sur ce réseau sur une ip fixe
docker run -d --name db --network app_network --ip 10.10.0.100 -e POSTGRES_PASSWORD=secret postgres:15

# Lancez l'application web sur le même réseau sur une ip dynamique
docker run -d --name webapp --network app_network -p 8080:80 -e DATABASE_HOST=db mon_app_web

# Vérifiez la communication - Depuis le conteneur webapp, pingez la DB par son nom
docker exec webapp ping -c 3 db
```

#### Commandes de diagnostic réseau
```bash
# Voir les réseaux d'un conteneur
docker inspect -f '{{json .NetworkSettings.Networks}}' mon_conteneur | jq  # jq est la commande pour indenter le retour json

# Tester la connectivité depuis un conteneur
docker exec mon_conteneur ping -c 3 autre_conteneur 
docker exec mon_conteneur nslookup autre_conteneur # attention, l'image doit contenir l'application nslookup, sinon il faut l'installer sur le conteneur

# Voir les règles iptables Docker
sudo iptables -L -n -t nat | grep -i docker

# Inspecter le bridge Docker
docker network inspect bridge

# Logs du daemon Docker (problèmes réseau)
journalctl -u docker.service | grep -i network
```