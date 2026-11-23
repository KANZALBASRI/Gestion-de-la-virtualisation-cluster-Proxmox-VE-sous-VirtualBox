# 08 – Migration des machines virtuelles

## 1. Objectif

L'objectif de cette étape est de tester la migration des machines virtuelles entre les deux nœuds du cluster Proxmox VE.

Grâce au stockage partagé NFS, les deux nœuds peuvent accéder aux mêmes disques virtuels. Cela facilite le déplacement des machines virtuelles sans devoir copier leurs disques locaux.

Deux migrations seront réalisées :

- Alpine-VM : PVE1 vers PVE2.
- Ubuntu-VM : PVE2 vers PVE1.

## 2. Vérification avant migration

Avant de commencer, vérifier que :

- Les deux nœuds Proxmox sont actifs.
- Le cluster fonctionne correctement.
- Le stockage NFS-Shared est accessible depuis PVE1 et PVE2.
- Les machines virtuelles sont correctement configurées.

Sur PVE1 et PVE2 :

```bash
pvecm status
pvecm nodes
pvesm status
```

Vérifier également les machines virtuelles :

```bash
qm list
```

Le stockage NFS doit apparaître avec le statut actif.

## 3. Migration d'Alpine-VM de PVE1 vers PVE2

Ouvrir l'interface Web Proxmox :

`https://192.168.122.27:8006`

Suivre les étapes :

1. Sélectionner PVE1.
2. Sélectionner Alpine-VM.
3. Vérifier que la VM est hébergée sur NFS-Shared.
4. Cliquer sur More.
5. Sélectionner Migrate.
6. Choisir PVE2 comme nœud de destination.
7. Vérifier les paramètres de migration.
8. Cliquer sur Migrate.

Attendre la fin de l'opération.

### Vérification

Sur PVE2, vérifier que la VM apparaît :

```bash
qm list
```

Vérifier sa configuration :

```bash
qm config VMID
```

Démarrer la VM si nécessaire et vérifier son fonctionnement.

## 4. Migration d'Ubuntu-VM de PVE2 vers PVE1

Ouvrir l'interface Web Proxmox.

Suivre les étapes :

1. Sélectionner PVE2.
2. Sélectionner Ubuntu-VM.
3. Cliquer sur More.
4. Sélectionner Migrate.
5. Choisir PVE1 comme destination.
6. Vérifier les paramètres.
7. Lancer la migration.

Attendre la fin de l'opération.

### Vérification

Sur PVE1 :

```bash
qm list
```

Vérifier la configuration :

```bash
qm config VMID
```

Vérifier que la machine virtuelle démarre correctement.

## 5. Vérification après migration

Pour chaque machine virtuelle, vérifier :

- Le démarrage du système.
- La connectivité réseau.
- L'accès SSH.
- Le fonctionnement de QEMU Guest Agent.
- L'accès au stockage partagé.

Tester la communication réseau :

Depuis Alpine :

```bash
ping IP_UBUNTU
```

Depuis Ubuntu :

```bash
ping IP_ALPINE
```

Tester également les connexions SSH.

## 6. Migration à froid et migration à chaud

Proxmox permet deux types de migration.

### Migration à froid (Offline Migration)

La machine virtuelle est arrêtée avant le déplacement.

Avantages :
- Procédure simple.
- Pas de service interrompu pendant la copie de la mémoire, puisqu'elle n'est pas transférée.

Inconvénient :
- La VM est indisponible pendant la migration.

### Migration à chaud (Live Migration)

La machine virtuelle reste en fonctionnement pendant la migration.

Avantages :
- Réduction de l'interruption des services.
- Utile pour la maintenance des nœuds.

Conditions :
- Stockage compatible et accessible depuis les deux nœuds.
- Réseau stable.
- Ressources CPU et mémoire suffisantes.
- Configuration compatible entre les nœuds.

## 7. Tests et validation

| Test | Résultat attendu |
|---|---|
| Alpine PVE1 → PVE2 | Migration réussie |
| Ubuntu PVE2 → PVE1 | Migration réussie |
| Démarrage après migration | Fonctionnel |
| Réseau après migration | Fonctionnel |
| SSH après migration | Fonctionnel |
| Accès NFS | Accessible |
| Migration à froid | Testée |
| Migration à chaud | Testée si possible |

## 8. Résultat final

À la fin de cette étape, les migrations des deux machines virtuelles doivent être vérifiées.

Le stockage NFS partagé permet aux deux nœuds d'accéder aux mêmes disques virtuels, facilitant ainsi la migration entre PVE1 et PVE2.

Cette fonctionnalité améliore la flexibilité de l'infrastructure et facilite les opérations de maintenance.
