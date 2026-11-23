# 09 – Configuration et étude de la haute disponibilité (HA)

## 1. Objectif

La haute disponibilité (High Availability – HA) est une fonctionnalité permettant de maintenir la disponibilité des services hébergés sur une infrastructure de virtualisation.

Dans Proxmox VE, HA permet de surveiller les machines virtuelles et de les redémarrer automatiquement sur un autre nœud lorsque le nœud qui les héberge devient indisponible, sous réserve que les conditions nécessaires soient réunies.

L'objectif de cette étape est de :
- Comprendre le fonctionnement de HA.
- Étudier le rôle du quorum.
- Vérifier les conditions nécessaires à son activation.
- Configurer une ressource HA.
- Étudier le comportement du cluster en cas de panne simulée.

## 2. Architecture

L'infrastructure utilisée :

| Machine | Fonction |
|---|---|
| PVE1 | Premier nœud Proxmox |
| PVE2 | Deuxième nœud Proxmox |
| STORAGE-VM | Serveur NFS partagé |
| Alpine-VM | VM hébergée dans le cluster |
| Ubuntu-VM | VM hébergée dans le cluster |

Les deux nœuds appartiennent au cluster `mycluser`.

Le stockage NFS partagé permet aux deux nœuds d'accéder aux mêmes disques virtuels.

## 3. Comprendre le quorum

Le quorum est un mécanisme permettant au cluster de déterminer s'il possède suffisamment de votes pour fonctionner de manière cohérente.

Il évite notamment que plusieurs parties d'un cluster prennent simultanément des décisions contradictoires.

Dans un cluster Proxmox à deux nœuds, chaque nœud possède normalement un vote.

Le quorum nécessite la majorité des votes disponibles.

Vérifier l'état du quorum :

```bash
pvecm status
```

Observer les informations suivantes :

- Nodes : nombre de nœuds.
- Expected votes : nombre de votes attendus.
- Total votes : nombre de votes disponibles.
- Quorum : nombre de votes nécessaires.
- Quorate : indique si le cluster possède le quorum.

## 4. Comprendre le QDevice

Le QDevice (Quorum Device) est un mécanisme permettant d'ajouter un vote externe au cluster.

Il est particulièrement utile pour les clusters composés de deux nœuds.

Un troisième système indépendant peut servir de QDevice afin d'aider le cluster à maintenir le quorum lorsqu'un des deux nœuds devient indisponible.

Dans ce projet, STORAGE-VM peut être étudié comme candidat pour héberger ce service, à condition de respecter les prérequis et de disposer d'une configuration réseau adaptée.

**Attention :** le QDevice améliore la gestion du quorum, mais ne constitue pas un mécanisme de stockage ni une garantie de disponibilité physique.

## 5. Vérification des conditions nécessaires à HA

Avant d'activer HA, vérifier les éléments suivants :

### 5.1 État du cluster

```bash
pvecm status
pvecm nodes
```

Les deux nœuds doivent être visibles et le cluster doit disposer du quorum.

### 5.2 État du stockage partagé

```bash
pvesm status
```

Vérifier que NFS-Shared est actif sur les deux nœuds.

### 5.3 État des machines virtuelles

```bash
qm list
```

Vérifier que les machines virtuelles sont correctement configurées.

### 5.4 Vérification des ressources

Vérifier que chaque nœud dispose de suffisamment de mémoire RAM, de CPU et d'espace de stockage pour accueillir les machines virtuelles susceptibles d'être redémarrées.

## 6. Configuration de HA dans Proxmox

Depuis l'interface Web :

1. Accéder à Datacenter.
2. Ouvrir la section HA.
3. Sélectionner Resources.
4. Cliquer sur Add.
5. Sélectionner une machine virtuelle.
6. Configurer les paramètres de la ressource.
7. Confirmer l'ajout.

La machine virtuelle apparaît alors dans la liste des ressources HA.

On peut également consulter les groupes HA pour définir les nœuds préférés et les règles de placement, selon les besoins.

**Remarque :** ne pas ajouter une VM à HA avant d'avoir vérifié le quorum, le stockage partagé et les ressources disponibles.

## 7. Vérification de l'état HA

Dans l'interface Web, accéder à :

Datacenter → HA → Status

Observer l'état des ressources et des services HA.

En ligne de commande :

```bash
ha-manager status
```

Cette commande permet de consulter l'état du gestionnaire HA et des ressources surveillées.

## 8. Simulation d'une panne

La simulation doit être réalisée dans un environnement de laboratoire contrôlé.

Avant de commencer :
- Vérifier que les données importantes sont sauvegardées.
- Vérifier que le cluster possède le quorum.
- Vérifier que le stockage NFS reste accessible.
- S'assurer que les deux nœuds disposent des ressources nécessaires.

Une simulation peut consister à rendre un nœud indisponible et à observer le comportement du gestionnaire HA.

Après la simulation, vérifier :

```bash
pvecm status
ha-manager status
```

Observer si la VM protégée par HA est redémarrée sur le nœud restant.

**Attention :** ne pas arrêter brutalement l'hôte Fedora 44. Les deux nœuds Proxmox fonctionnent sur cet hôte et seraient donc arrêtés simultanément.

## 9. Limites de la haute disponibilité dans ce projet

L'infrastructure présente certaines limites :

- Les deux nœuds Proxmox sont hébergés sur un seul ordinateur physique.
- Une panne de Fedora 44 entraîne l'arrêt des deux nœuds.
- Le stockage NFS dépend également de la disponibilité de STORAGE-VM et de son hôte.
- Un cluster à deux nœuds nécessite une attention particulière concernant le quorum.

Par conséquent, cette infrastructure permet d'étudier les mécanismes HA, mais ne représente pas une architecture de production entièrement tolérante aux pannes.

## 10. Tests et validation

| Vérification | Résultat attendu |
|---|---|
| État du cluster | Deux nœuds visibles |
| Quorum | Actif |
| Stockage NFS | Accessible |
| Ressources HA | Configurées |
| État du gestionnaire HA | Vérifié |
| Redémarrage automatique | Étudié ou testé |
| Limites de l'infrastructure | Identifiées |

## 11. Résultat final

À la fin de cette étape, le fonctionnement de la haute disponibilité dans Proxmox VE est étudié.

Le projet permet de comprendre le rôle du quorum, du QDevice, du stockage partagé et du gestionnaire HA.

Les limites liées à l'utilisation de deux nœuds virtualisés sur un même hôte physique sont également identifiées.

Cette étape prépare la validation finale de l'infrastructure.
