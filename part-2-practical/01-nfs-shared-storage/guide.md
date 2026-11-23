# 07 – Configuration du stockage partagé NFS

## 1. Objectif

L'objectif de cette étape est de mettre en place un stockage partagé entre les deux nœuds Proxmox VE à l'aide du protocole NFS (Network File System).

Un serveur dédié, nommé STORAGE-VM, sera utilisé pour centraliser le stockage des machines virtuelles.

Cette configuration permettra à PVE1 et PVE2 d'accéder aux mêmes fichiers et disques virtuels, facilitant ainsi la migration des machines virtuelles entre les deux nœuds.

### Architecture

| Machine | Fonction |
|---|---|
| PVE1 | Premier nœud Proxmox |
| PVE2 | Deuxième nœud Proxmox |
| STORAGE-VM | Serveur NFS |
| Alpine-VM | Machine virtuelle invitée |
| Ubuntu-VM | Machine virtuelle invitée |

Adresses des nœuds Proxmox :

- PVE1 : `192.168.122.27`
- PVE2 : `192.168.122.37`

L'adresse IP de STORAGE-VM sera notée `IP_STORAGE`.

## 2. Préparation du serveur STORAGE-VM

Démarrer STORAGE-VM et ouvrir une session dans son terminal.

Vérifier la configuration réseau :

```bash
ip -br addr
ip route
hostname
```

Identifier l'adresse IP du serveur :

```bash
hostname -I
```

Vérifier que les deux nœuds Proxmox sont accessibles :

```bash
ping 192.168.122.27
ping 192.168.122.37
```

**Résultat attendu :** STORAGE-VM communique avec PVE1 et PVE2.

## 3. Installation du serveur NFS

Sur STORAGE-VM, installer les outils NFS.

Pour une distribution Fedora ou compatible RHEL :

```bash
sudo dnf update -y
sudo dnf install nfs-utils -y
```

Activer le service NFS :

```bash
sudo systemctl enable --now nfs-server
```

Vérifier son état :

```bash
sudo systemctl status nfs-server
```

Le service doit être actif.

## 4. Création du répertoire partagé

Créer un répertoire destiné à accueillir les fichiers des machines virtuelles :

```bash
sudo mkdir -p /srv/nfs/proxmox
```

Configurer les permissions :

```bash
sudo chmod 777 /srv/nfs/proxmox
```

Vérifier :

```bash
ls -ld /srv/nfs/proxmox
```

Ce répertoire sera utilisé comme espace de stockage partagé par les deux nœuds.

## 5. Configuration des exports NFS

Modifier le fichier de configuration :

```bash
sudo nano /etc/exports
```

Ajouter la ligne suivante :

```text
/srv/nfs/proxmox 192.168.122.0/24(rw,sync,no_subtree_check,no_root_squash)
```

Explication :

- `/srv/nfs/proxmox` : répertoire partagé.
- `192.168.122.0/24` : réseau autorisé.
- `rw` : autorise la lecture et l'écriture.
- `sync` : écrit les modifications de manière synchrone.
- `no_subtree_check` : désactive certaines vérifications du système de fichiers.
- `no_root_squash` : conserve les privilèges root des clients NFS.

**Attention :** cette configuration est adaptée à un laboratoire isolé. `no_root_squash` donne des privilèges élevés aux clients autorisés et ne doit pas être utilisé sans évaluation des risques dans un environnement de production.

Appliquer la configuration :

```bash
sudo exportfs -rav
```

Vérifier les exports :

```bash
sudo exportfs -v
```

## 6. Configuration du pare-feu

Si Firewalld est actif sur STORAGE-VM, autoriser les services NFS :

```bash
sudo firewall-cmd --permanent --add-service=nfs
sudo firewall-cmd --permanent --add-service=mountd
sudo firewall-cmd --permanent --add-service=rpc-bind
sudo firewall-cmd --reload
```

Vérifier les règles :

```bash
sudo firewall-cmd --list-services
```

## 7. Vérification du serveur NFS

Afficher les répertoires exportés :

```bash
sudo exportfs -v
```

Vérifier les services :

```bash
sudo systemctl status nfs-server
```

Le serveur NFS est maintenant prêt à recevoir les connexions des nœuds Proxmox.

## 8. Ajout du stockage NFS dans Proxmox

Ouvrir l'interface Web de PVE1 :

`https://192.168.122.27:8006`

Dans l'interface :

1. Sélectionner `Datacenter`.
2. Ouvrir `Storage`.
3. Cliquer sur `Add`.
4. Sélectionner `NFS`.

Renseigner les paramètres :

| Paramètre | Valeur |
|---|---|
| ID | NFS-Shared |
| Server | IP_STORAGE |
| Export | /srv/nfs/proxmox |
| Content | Disk image |
| Nodes | PVE1 et PVE2 |
| Enable | Activé |

Cliquer sur `Add`.

Le stockage doit apparaître dans l'arborescence du Datacenter.

**Important :** vérifier que les deux nœuds sont sélectionnés pour utiliser ce stockage. Le cluster partage la configuration du stockage entre ses nœuds.

## 9. Vérification du stockage

Sur PVE1 et PVE2, vérifier les stockages disponibles :

```bash
pvesm status
```

Vérifier que `NFS-Shared` apparaît comme actif.

On peut également vérifier le montage NFS :

```bash
mount | grep nfs
```

Ou :

```bash
df -h
```

**Résultat attendu :** les deux nœuds Proxmox accèdent au même stockage NFS.

## 10. Vérification de l'accès en écriture

Créer un fichier de test depuis PVE1 :

```bash
touch /mnt/pve/NFS-Shared/test-pve1.txt
```

Depuis PVE2, vérifier sa présence :

```bash
ls -l /mnt/pve/NFS-Shared/
```

Tester également l'écriture depuis PVE2 :

```bash
touch /mnt/pve/NFS-Shared/test-pve2.txt
```

Vérifier depuis PVE1 :

```bash
ls -l /mnt/pve/NFS-Shared/
```

Si les deux fichiers sont visibles, le partage fonctionne dans les deux sens.

## 11. Résultat final

À la fin de cette étape :

- STORAGE-VM fonctionne comme serveur NFS.
- Le répertoire `/srv/nfs/proxmox` est exporté.
- PVE1 et PVE2 utilisent le stockage `NFS-Shared`.
- Les deux nœuds peuvent lire et écrire dans le répertoire partagé.
- Le stockage est disponible pour accueillir les disques des machines virtuelles.

Ce stockage partagé sera utilisé dans l'étape suivante pour effectuer les migrations des machines virtuelles entre PVE1 et PVE2.

## 12. Commandes essentielles

| Fonction | Commande |
|---|---|
| Installer NFS | `sudo dnf install nfs-utils -y` |
| Activer NFS | `sudo systemctl enable --now nfs-server` |
| Créer le répertoire | `sudo mkdir -p /srv/nfs/proxmox` |
| Configurer les exports | `sudo nano /etc/exports` |
| Appliquer les exports | `sudo exportfs -rav` |
| Vérifier les exports | `sudo exportfs -v` |
| Vérifier le stockage | `pvesm status` |
| Vérifier le montage | `df -h` |
