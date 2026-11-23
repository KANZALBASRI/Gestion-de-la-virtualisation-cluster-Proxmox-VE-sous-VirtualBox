# Référence des commandes – Projet Proxmox VE

Ce document regroupe les principales commandes utilisées dans le projet de gestion de la virtualisation avec Proxmox VE.

**Environnement :**
- Hôte : Fedora 44.
- Hyperviseur externe : Virtual Machine Manager.
- PVE1 : `192.168.122.27`.
- PVE2 : `192.168.122.37`.
- Cluster : `mycluser`.
- Serveur NFS : STORAGE-VM.
- Machines virtuelles : Alpine Linux et Ubuntu Server.

---

## 1. Informations système

Afficher le nom de la machine :

```bash
hostname
hostnamectl
```

Afficher les informations système :

```bash
uname -a
cat /etc/os-release
```

Afficher les adresses IP :

```bash
ip addr
ip -br addr
```

Afficher les interfaces réseau :

```bash
ip link
```

Afficher les routes :

```bash
ip route
```

## 2. Tests réseau

Tester la communication entre les nœuds :

```bash
ping 192.168.122.27
ping 192.168.122.37
```

Afficher les connexions réseau :

```bash
ss -tulnp
```

Tester une connexion SSH :

```bash
ssh root@IP_ADDRESS
```

Vérifier la résolution des noms :

```bash
cat /etc/hosts
getent hosts pve1
getent hosts pve2
```

## 3. Administration du cluster Proxmox

Créer un cluster sur PVE1 :

```bash
pvecm create mycluser
```

Ajouter PVE2 au cluster :

```bash
pvecm add 192.168.122.27
```

Afficher l'état du cluster :

```bash
pvecm status
```

Afficher les nœuds :

```bash
pvecm nodes
```

Afficher les informations Corosync :

```bash
corosync-cfgtool -s
```

Afficher les services du cluster :

```bash
systemctl status corosync
systemctl status pve-cluster
```

## 4. Administration des machines virtuelles

Afficher toutes les VM :

```bash
qm list
```

Afficher la configuration d'une VM :

```bash
qm config VMID
```

Démarrer une VM :

```bash
qm start VMID
```

Arrêter une VM :

```bash
qm shutdown VMID
```

Arrêter immédiatement une VM :

```bash
qm stop VMID
```

Redémarrer une VM :

```bash
qm reboot VMID
```

Afficher son état :

```bash
qm status VMID
```

## 5. Administration du stockage

Afficher les stockages Proxmox :

```bash
pvesm status
```

Afficher les disques :

```bash
lsblk
```

Afficher l'espace disponible :

```bash
df -h
```

Afficher les systèmes de fichiers montés :

```bash
mount
```

Vérifier le montage NFS :

```bash
mount | grep nfs
```

## 6. Administration du serveur NFS

Installer les outils NFS sur Fedora :

```bash
sudo dnf install nfs-utils -y
```

Activer le serveur :

```bash
sudo systemctl enable --now nfs-server
```

Vérifier le service :

```bash
sudo systemctl status nfs-server
```

Créer un répertoire partagé :

```bash
sudo mkdir -p /srv/nfs/proxmox
```

Modifier les exports :

```bash
sudo nano /etc/exports
```

Appliquer les exports :

```bash
sudo exportfs -rav
```

Afficher les exports :

```bash
sudo exportfs -v
```

## 7. Migration des machines virtuelles

La migration peut être effectuée depuis l'interface Web Proxmox :

`VM → More → Migrate`

Vérifier les VM :

```bash
qm list
```

Vérifier le stockage :

```bash
pvesm status
```

Vérifier la configuration d'une VM :

```bash
qm config VMID
```

## 8. Haute disponibilité (HA)

Afficher l'état HA :

```bash
ha-manager status
```

Vérifier le quorum :

```bash
pvecm status
```

Vérifier les nœuds :

```bash
pvecm nodes
```

Consulter les services HA :

```bash
systemctl status pve-ha-lrm
systemctl status pve-ha-crm
```

## 9. Configuration des machines virtuelles invitées

### Alpine Linux

Mettre à jour les paquets :

```bash
apk update
apk upgrade
```

Installer SSH :

```bash
apk add openssh
```

Activer SSH :

```bash
rc-update add sshd default
rc-service sshd start
```

Vérifier SSH :

```bash
rc-service sshd status
```

Installer QEMU Guest Agent :

```bash
apk add qemu-guest-agent
rc-update add qemu-guest-agent default
rc-service qemu-guest-agent start
```

### Ubuntu Server

Mettre à jour le système :

```bash
sudo apt update
sudo apt upgrade -y
```

Installer SSH :

```bash
sudo apt install openssh-server -y
```

Activer SSH :

```bash
sudo systemctl enable --now ssh
```

Installer QEMU Guest Agent :

```bash
sudo apt install qemu-guest-agent -y
sudo systemctl enable --now qemu-guest-agent
```

## 10. Vérification des services

Sur Proxmox :

```bash
systemctl status pve-cluster
systemctl status corosync
systemctl status pvedaemon
systemctl status pveproxy
```

Sur Ubuntu :

```bash
sudo systemctl status ssh
sudo systemctl status qemu-guest-agent
```

Sur Alpine :

```bash
rc-service sshd status
rc-service qemu-guest-agent status
```

## 11. Dépannage

Afficher les derniers messages système :

```bash
journalctl -xe
```

Afficher les journaux d'un service :

```bash
journalctl -u SERVICE_NAME
```

Afficher les dernières lignes :

```bash
journalctl -n 50
```

Vérifier les ports en écoute :

```bash
ss -tulnp
```

Vérifier l'espace disque :

```bash
df -h
```

Vérifier la mémoire :

```bash
free -h
```

---

## 12. Tableau récapitulatif

| Fonction | Commande principale |
|---|---|
| Informations système | `hostnamectl` |
| Configuration réseau | `ip -br addr` |
| Test réseau | `ping IP` |
| État du cluster | `pvecm status` |
| Liste des nœuds | `pvecm nodes` |
| Liste des VM | `qm list` |
| Configuration VM | `qm config VMID` |
| État du stockage | `pvesm status` |
| État HA | `ha-manager status` |
| État NFS | `systemctl status nfs-server` |
| Journaux système | `journalctl -xe` |

**Remarque :** remplacer `VMID`, `IP_ADDRESS` et `SERVICE_NAME` par les valeurs correspondant à l'environnement réel.
