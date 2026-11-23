# 05 – Création du cluster Proxmox VE

## 1. Objectif

L'objectif de cette étape est de regrouper les deux serveurs Proxmox VE dans un cluster afin de les administrer depuis une seule interface Web.

Le cluster sera composé de :

- PVE1 : premier nœud Proxmox VE.
- PVE2 : deuxième nœud Proxmox VE.

Les autres machines ne sont pas des nœuds du cluster :

- STORAGE-VM : serveur Ubuntu externe au cluster, destiné à NFS et QDevice.
- Alpine Linux : machine virtuelle hébergée dans Proxmox.
- Ubuntu Server : machine virtuelle hébergée dans Proxmox.

Nom du cluster : `mycluser`

## 2. Architecture

```text
             Windows 11
                  |
              VirtualBox
                  |
       +----------+----------+
       |                     |
      PVE1                  PVE2
  192.168.56.11         192.168.56.12
  10.10.10.11           10.10.10.12
       |                     |
       +----------+----------+
                  |
           Cluster Proxmox
               mycluser
                  |
        +---------+---------+
        |                   |
     Alpine VM           Ubuntu VM

       STORAGE-VM
       192.168.56.20
       10.10.10.20
       (NFS + QDevice)
```

Le réseau de gestion `192.168.56.0/24` sera utilisé pour la communication Corosync dans cette configuration. Le réseau interne `10.10.10.0/24` sera réservé aux échanges internes, notamment au stockage NFS et aux machines invitées.

## 3. Prérequis

Avant de créer le cluster, vérifier les conditions suivantes :

- [ ] Proxmox VE est installé sur PVE1 et PVE2.
- [ ] Les deux nœuds utilisent des versions compatibles de Proxmox VE.
- [ ] Les deux nœuds possèdent des hostnames différents.
- [ ] Les adresses IP sont statiques.
- [ ] Les deux nœuds communiquent correctement.
- [ ] Les noms des nœuds sont correctement résolus.
- [ ] Les horloges des deux machines sont synchronisées.
- [ ] Le réseau utilisé par Corosync est stable.

### Attention : machines virtuelles déjà installées

Un nœud qui rejoint un cluster ne doit pas contenir de machines virtuelles ou de conteneurs existants. Sa configuration `/etc/pve` est remplacée par celle du cluster.

Dans notre projet, Ubuntu Server est déjà installé comme VM sur PVE2. Il faut donc effectuer une sauvegarde de cette VM avant de joindre PVE2 au cluster, puis la restaurer après l'intégration.

Si le cluster est déjà créé et que PVE2 l'a déjà rejoint, ne répéter aucune commande de création ou d'ajout. Passer directement aux vérifications.

## 4. Vérification du réseau

### 4.1 Sur PVE1

Ouvrir le terminal de PVE1 et exécuter :

```bash
hostnamectl
ip -br addr
ip route
```

Tester la communication avec PVE2 :

```bash
ping -c 4 192.168.56.12
```

Tester également le réseau interne :

```bash
ping -c 4 10.10.10.12
```

### 4.2 Sur PVE2

Exécuter :

```bash
hostnamectl
ip -br addr
ip route
```

Tester la communication avec PVE1 :

```bash
ping -c 4 192.168.56.11
```

Tester également le réseau interne :

```bash
ping -c 4 10.10.10.11
```

**Résultat attendu :** les deux nœuds communiquent sur les deux réseaux.

## 5. Vérification des noms des nœuds

Sur PVE1 et PVE2, afficher le fichier :

```bash
cat /etc/hosts
```

Il doit contenir les correspondances suivantes :

```text
192.168.56.11 pve1.proxmox.lab pve1
192.168.56.12 pve2.proxmox.lab pve2
192.168.56.20 storage.proxmox.lab storage
```

Vérifier la résolution :

```bash
getent hosts pve1.proxmox.lab
getent hosts pve2.proxmox.lab
```

Les noms doivent correspondre aux bonnes adresses IP.

## 6. Sauvegarde de la VM présente sur PVE2

Cette étape concerne la VM Ubuntu Server hébergée sur PVE2.

Dans l'interface Web de PVE2 :

1. Sélectionner la VM Ubuntu Server.
2. Vérifier son identifiant (VMID).
3. Aller dans Backup.
4. Cliquer sur Backup now.
5. Sélectionner le stockage `local`.
6. Choisir le mode `Stop` pour effectuer une sauvegarde avec la VM arrêtée.
7. Lancer la sauvegarde.

Vérifier que le fichier de sauvegarde a bien été créé.

Depuis le terminal de PVE2, il est également possible de consulter les sauvegardes :

```bash
ls -lh /var/lib/vz/dump/
```

Conserver cette sauvegarde jusqu'à ce que la VM ait été restaurée et testée après la création du cluster.

## 7. Création du cluster sur PVE1

**Machine concernée : PVE1**

La création du cluster s'effectue uniquement sur le premier nœud.

Ouvrir le terminal de PVE1 et exécuter :

```bash
pvecm create mycluser --link0 192.168.56.11
```

Explication :

- `pvecm` : outil de gestion du cluster Proxmox.
- `create` : crée un nouveau cluster.
- `mycluser` : nom du cluster.
- `--link0` : définit l'adresse utilisée pour le premier lien de communication Corosync.

Attendre la fin de l'opération.

Vérifier l'état du cluster :

```bash
pvecm status
```

À ce stade, PVE1 doit être le premier nœud du cluster.

## 8. Ajouter PVE2 au cluster

**Machine concernée : PVE2**

Ouvrir le terminal de PVE2.

Exécuter :

```bash
pvecm add 192.168.56.11 --link0 192.168.56.12
```

Explication :

- `pvecm add` : ajoute le nœud actuel à un cluster existant.
- `192.168.56.11` : adresse de PVE1, déjà membre du cluster.
- `--link0 192.168.56.12` : adresse locale de PVE2 pour la communication Corosync.

Le système demande le mot de passe root de PVE1 et la confirmation de l'empreinte du certificat.

Confirmer uniquement si l'empreinte correspond bien au serveur attendu.

Attendre la fin de l'opération.

**Attention :** cette commande ne doit être exécutée qu'une seule fois pour joindre PVE2 au cluster.

## 9. Vérification du cluster

### 9.1 Depuis PVE1

Exécuter :

```bash
pvecm status
```

Puis :

```bash
pvecm nodes
```

### 9.2 Depuis PVE2

Exécuter les mêmes commandes :

```bash
pvecm status
pvecm nodes
```

Le résultat doit montrer les deux nœuds :

- pve1
- pve2

Les deux nœuds doivent appartenir au même cluster.

### 9.3 Vérification depuis l'interface Web

Ouvrir dans le navigateur Windows :

```text
https://192.168.56.11:8006
```

Se connecter avec le compte root.

Dans le panneau de gauche :

1. Sélectionner Datacenter.
2. Ouvrir Cluster.
3. Vérifier que les deux nœuds apparaissent.
4. Vérifier leur état.

Les deux nœuds doivent être visibles dans la même interface Proxmox.

## 10. Restaurer la VM Ubuntu Server

Après l'intégration de PVE2, restaurer la sauvegarde effectuée précédemment.

Dans l'interface Web :

1. Sélectionner le stockage local de PVE2.
2. Ouvrir Backups.
3. Sélectionner la sauvegarde Ubuntu Server.
4. Cliquer sur Restore.
5. Choisir un VMID disponible.
6. Sélectionner le stockage de destination.
7. Lancer la restauration.

Démarrer la VM et vérifier :

- Le démarrage du système.
- La configuration réseau.
- La connectivité.
- L'accès SSH si celui-ci est configuré.

La VM Alpine Linux hébergée sur PVE1 n'a pas besoin d'être déplacée pour cette opération.

## 11. Comprendre le quorum

Le quorum est le mécanisme qui permet au cluster de déterminer s'il dispose d'un nombre suffisant de votes pour fonctionner normalement.

Dans un cluster composé de deux nœuds :

| Élément | Valeur |
|---|---:|
| Nombre de nœuds | 2 |
| Nombre de votes | 2 |
| Quorum nécessaire | 2 |

Si PVE1 ou PVE2 devient indisponible, le nœud restant risque de perdre le quorum.

C'est pourquoi Proxmox permet d'ajouter un troisième vote avec un QDevice.

Dans notre projet, STORAGE-VM accueillera ultérieurement le service Corosync QNetd. Il permettra d'apporter un vote supplémentaire au cluster.

La configuration du QDevice sera réalisée dans une étape séparée.

Ne pas utiliser `pvecm expected 1` comme solution permanente pour contourner le quorum : cela ne remplace pas un troisième vote indépendant.

## 12. Dépannage

### Problème 1 : erreur de connexion pendant l'ajout de PVE2

Vérifier la connectivité :

```bash
ping -c 4 192.168.56.11
```

Vérifier également la résolution des noms :

```bash
getent hosts pve1.proxmox.lab
```

### Problème 2 : le nœud possède déjà une configuration de cluster

Afficher :

```bash
pvecm status
```

Ne pas tenter de recréer le cluster sans avoir identifié l'état actuel du nœud.

### Problème 3 : problème de quorum

Sur l'un des nœuds :

```bash
pvecm status
```

Examiner notamment :

- Expected votes.
- Total votes.
- Quorum.
- Membership information.

Avec deux nœuds, les deux doivent normalement être présents pour obtenir le quorum.

### Problème 4 : les deux nœuds n'apparaissent pas dans l'interface Web

Vérifier :

```bash
pvecm nodes
systemctl status corosync
systemctl status pve-cluster
```

Consulter les journaux en cas d'erreur :

```bash
journalctl -u corosync -n 50
```

Ne pas supprimer manuellement les fichiers de configuration du cluster pour tenter de corriger le problème.

## 13. Résultats attendus

| Vérification | Résultat attendu |
|---|---|
| Communication PVE1/PVE2 | Réussie |
| Création du cluster | Réussie |
| Ajout de PVE2 | Réussi |
| Nom du cluster | mycluser |
| Nombre de nœuds | 2 |
| État du cluster | Quorate lorsque les deux nœuds sont disponibles |
| Interface Web | Deux nœuds visibles |
| VM Ubuntu Server | Restaurée et fonctionnelle |
| VM Alpine Linux | Toujours fonctionnelle |

## 14. Conclusion

Le cluster Proxmox VE à deux nœuds est maintenant créé. PVE1 et PVE2 peuvent être administrés depuis une seule interface Web.

Cette configuration permet de préparer les opérations de stockage partagé et de migration des machines virtuelles.

Le cluster possède toutefois une limitation liée au quorum : avec seulement deux nœuds, la perte d'un nœud peut empêcher le fonctionnement normal du cluster. L'ajout d'un QDevice sur STORAGE-VM sera donc nécessaire pour poursuivre les tests de haute disponibilité.

**Prochaine étape :** configuration du stockage partagé NFS entre les deux nœuds Proxmox.
