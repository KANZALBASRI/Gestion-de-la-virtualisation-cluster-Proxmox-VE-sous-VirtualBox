# 09 – Configuration et test de la haute disponibilité (HA)

## 1. Objectif

La haute disponibilité (High Availability – HA) est une fonctionnalité permettant de maintenir la disponibilité des machines virtuelles lorsqu'un nœud Proxmox devient indisponible.

Dans un cluster Proxmox VE, le gestionnaire HA surveille les ressources configurées. Lorsqu'un nœud tombe en panne, les machines virtuelles concernées peuvent être redémarrées automatiquement sur un autre nœud disponible.

Les objectifs de cette étape sont :
- Comprendre le fonctionnement de HA.
- Vérifier les conditions nécessaires à son activation.
- Comprendre le rôle du quorum et du QDevice.
- Configurer une ressource HA.
- Tester le fonctionnement de la haute disponibilité.

## 2. Architecture du projet

| Composant | Fonction |
|---|---|
| PVE1 | Premier nœud Proxmox |
| PVE2 | Deuxième nœud Proxmox |
| STORAGE-VM | Serveur NFS |
| Alpine-VM | VM Linux |
| Ubuntu-VM | VM Linux |
| Cluster | mycluser |

Les deux nœuds utilisent le stockage partagé NFS-Shared pour accéder aux disques des machines virtuelles.

## 3. Prérequis de HA

Avant d'activer la haute disponibilité, vérifier :

- Les deux nœuds sont opérationnels.
- Le cluster possède le quorum.
- Le stockage partagé est accessible depuis les deux nœuds.
- Les ressources CPU et RAM sont suffisantes.
- Les machines virtuelles sont correctement configurées.
- Le mécanisme de watchdog utilisé par Proxmox est opérationnel.

Commandes de vérification :

```bash
pvecm status
pvecm nodes
pvesm status
qm list
ha-manager status
```

## 4. Comprendre le quorum et le QDevice

Le quorum permet au cluster de déterminer s'il possède suffisamment de votes pour prendre des décisions.

Dans un cluster de deux nœuds, chaque nœud possède normalement un vote. La perte d'un nœud peut donc entraîner la perte du quorum.

Pour améliorer cette situation, il est recommandé d'ajouter un QDevice.

Le QDevice est un troisième vote fourni par un système externe au cluster.

Il permet notamment à un cluster à deux nœuds de conserver le quorum lorsqu'un seul nœud reste disponible.

**Remarque :** le QDevice doit être installé et configuré sur un système indépendant des deux nœuds Proxmox. STORAGE-VM peut être étudié comme candidat, mais s'il dépend du même hôte physique, il ne constitue pas une véritable protection contre la panne de cet hôte.

## 5. Configuration des ressources HA

Depuis l'interface Web Proxmox :

1. Accéder à Datacenter.
2. Ouvrir HA.
3. Sélectionner Resources.
4. Cliquer sur Add.
5. Sélectionner la machine virtuelle à protéger.
6. Définir les paramètres de la ressource.
7. Confirmer l'ajout.

Exemple : ajouter Alpine-VM comme ressource HA.

La VM doit utiliser un stockage accessible depuis les deux nœuds.

Répéter l'opération pour Ubuntu-VM si les ressources disponibles le permettent.

## 6. Vérification de l'état HA

Dans l'interface Web :

Datacenter → HA → Status

En ligne de commande :

```bash
ha-manager status
```

Vérifier :
- L'état des ressources HA.
- Le nœud qui héberge chaque VM.
- L'état du gestionnaire HA.
- Les éventuelles erreurs.

## 7. Test de la haute disponibilité

Le test consiste à vérifier le comportement d'une machine virtuelle lorsque son nœud devient indisponible.

### Préparation

Avant le test :

- Vérifier le quorum.
- Vérifier le fonctionnement du QDevice s'il est configuré.
- Vérifier que NFS-Shared est accessible depuis les deux nœuds.
- Vérifier que le nœud de destination possède suffisamment de ressources.
- Sauvegarder les données importantes.

### Simulation contrôlée

Dans un environnement de laboratoire, effectuer une simulation de panne d'un nœud uniquement après avoir vérifié les mécanismes de quorum et de fencing.

Observer le comportement de la VM protégée par HA.

Vérifier ensuite :

```bash
pvecm status
ha-manager status
qm list
```

**Résultat attendu :** si les conditions HA sont réunies, la VM peut être redémarrée automatiquement sur le nœud disponible.

Attention : une migration manuelle ou une maintenance planifiée ne constitue pas à elle seule un test de basculement automatique après panne.

Ne pas arrêter brutalement Fedora 44 : les deux nœuds Proxmox dépendent du même hôte physique.

## 8. Limites de l'infrastructure

Cette infrastructure présente plusieurs limites :

- Les deux nœuds Proxmox fonctionnent sur le même ordinateur physique.
- Une panne de Fedora 44 entraîne l'arrêt des deux nœuds.
- STORAGE-VM constitue un point de défaillance si le stockage NFS devient indisponible.
- Le quorum nécessite une attention particulière dans un cluster à deux nœuds.
- La capacité de redémarrage d'une VM dépend des ressources disponibles sur le nœud restant.

Une infrastructure de production nécessite notamment des hôtes physiques indépendants, un stockage fiable et une configuration de quorum adaptée.

## 9. Résultat final

À la fin de cette étape, la haute disponibilité est étudiée et, si les conditions le permettent, configurée et testée.

Le projet permet de comprendre le fonctionnement du gestionnaire HA, le rôle du quorum et l'importance du stockage partagé dans une infrastructure Proxmox VE.
