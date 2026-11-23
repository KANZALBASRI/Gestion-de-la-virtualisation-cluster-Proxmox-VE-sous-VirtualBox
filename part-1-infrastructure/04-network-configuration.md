# 04 – Configuration réseau

## 1. Objectif

L'objectif de cette étape est de configurer et de vérifier la communication réseau entre les trois machines virtuelles créées dans VirtualBox :

- PVE1 : premier nœud Proxmox VE.
- PVE2 : deuxième nœud Proxmox VE.
- STORAGE-VM : serveur Ubuntu utilisé pour NFS et QDevice.

Nous allons également préparer le réseau des machines virtuelles Alpine Linux et Ubuntu Server créées à l'intérieur de Proxmox.

À la fin de cette étape, les trois machines VirtualBox devront communiquer entre elles, et les deux nœuds Proxmox devront pouvoir communiquer sur leur réseau interne.

## 2. Architecture réseau

Le laboratoire utilise deux réseaux VirtualBox distincts.

### 2.1 Réseau Host-only

Ce réseau permet à Windows 11 de communiquer avec les machines VirtualBox.

Réseau : `192.168.56.0/24`

| Machine | Adresse IP |
|---|---|
| Windows 11 | 192.168.56.1 |
| PVE1 | 192.168.56.11 |
| PVE2 | 192.168.56.12 |
| STORAGE-VM | 192.168.56.20 |

### 2.2 Réseau Internal Network

Ce réseau permet aux trois machines VirtualBox de communiquer entre elles, sans être directement accessible depuis Windows.

Nom VirtualBox : `intnet-proxmox`

Réseau : `10.10.10.0/24`

| Machine | Adresse IP |
|---|---|
| PVE1 | 10.10.10.11 |
| PVE2 | 10.10.10.12 |
| STORAGE-VM | 10.10.10.20 |

### 2.3 Schéma général

```text
Windows 11
192.168.56.1
     |
     | Host-only
     |
     +-------------------+
     |                   |
    PVE1                PVE2             STORAGE-VM
 .56.11 / .10.11    .56.12 / .10.12    .56.20 / .10.20
     |                   |                   |
     +-------------------+-------------------+
                 Internal Network
                   10.10.10.0/24
```

## 3. Vérification des cartes réseau dans VirtualBox

Avant de modifier la configuration réseau, vérifier les paramètres de chaque machine.

Pour PVE1, PVE2 et STORAGE-VM :

1. Éteindre la machine virtuelle.
2. Ouvrir VirtualBox.
3. Sélectionner la machine.
4. Aller dans Configuration → Réseau.

Les paramètres doivent être les suivants :

| Adaptateur | Mode | Configuration |
|---|---|---|
| Adaptateur 1 | Host-only Adapter | Réseau Host-only créé précédemment |
| Adaptateur 2 | Internal Network | `intnet-proxmox` |

Pour les deux machines Proxmox, vérifier également que le mode Promiscuous est configuré sur Allow All si nécessaire pour le trafic des machines invitées.

Démarrer les trois machines après vérification.

## 4. Identifier les interfaces réseau

Les noms des interfaces peuvent varier selon la machine. Il ne faut donc pas supposer qu'elles s'appellent toujours `enp0s3` et `enp0s8`.

Sur PVE1 et PVE2, exécuter :

```bash
ip -br link
ip -br addr
```

Sur STORAGE-VM, exécuter :

```bash
ip -br link
ip -br addr
```

Identifier les deux interfaces correspondant aux deux adaptateurs VirtualBox :

- Interface Host-only.
- Interface Internal Network.

Pour éviter toute confusion, comparer les adresses MAC affichées par Linux avec celles de VirtualBox.

## 5. Configuration réseau de PVE1

**Machine concernée : PVE1**

### 5.1 Principe

Proxmox utilise des bridges Linux pour connecter les machines virtuelles au réseau physique ou virtuel.

Nous allons utiliser :

- `vmbr0` : bridge du réseau Host-only.
- `vmbr1` : bridge du réseau Internal Network.

Le bridge `vmbr0` est normalement déjà créé pendant l'installation de Proxmox.

### 5.2 Sauvegarder la configuration

Avant toute modification :

```bash
cp /etc/network/interfaces /etc/network/interfaces.backup
```

Afficher la configuration actuelle :

```bash
cat /etc/network/interfaces
```

### 5.3 Modifier la configuration

Ouvrir le fichier :

```bash
nano /etc/network/interfaces
```

Conserver les noms réels des interfaces identifiées précédemment.

Exemple de configuration, en remplaçant `INTERFACE_HOSTONLY` et `INTERFACE_INTERNAL` par les vrais noms :

```bash
auto lo
iface lo inet loopback

iface INTERFACE_HOSTONLY inet manual

iface INTERFACE_INTERNAL inet manual

auto vmbr0
iface vmbr0 inet static
    address 192.168.56.11/24
    bridge-ports INTERFACE_HOSTONLY
    bridge-stp off
    bridge-fd 0

auto vmbr1
iface vmbr1 inet static
    address 10.10.10.11/24
    bridge-ports INTERFACE_INTERNAL
    bridge-stp off
    bridge-fd 0

source /etc/network/interfaces.d/*
```

Ne pas recopier littéralement les noms génériques des interfaces.

### 5.4 Appliquer la configuration

Il est préférable d'effectuer cette opération depuis la console VirtualBox, afin d'éviter de perdre une connexion SSH.

```bash
ifreload -a
```

Vérifier les adresses :

```bash
ip -br addr
```

Résultat attendu : `vmbr0` doit avoir l'adresse `192.168.56.11` et `vmbr1` l'adresse `10.10.10.11`.

## 6. Configuration réseau de PVE2

**Machine concernée : PVE2**

Répéter la même procédure que pour PVE1.

Sauvegarder la configuration :

```bash
cp /etc/network/interfaces /etc/network/interfaces.backup
```

Afficher le fichier :

```bash
cat /etc/network/interfaces
```

Modifier le fichier :

```bash
nano /etc/network/interfaces
```

Exemple de configuration :

```bash
auto lo
iface lo inet loopback

iface INTERFACE_HOSTONLY inet manual

iface INTERFACE_INTERNAL inet manual

auto vmbr0
iface vmbr0 inet static
    address 192.168.56.12/24
    bridge-ports INTERFACE_HOSTONLY
    bridge-stp off
    bridge-fd 0

auto vmbr1
iface vmbr1 inet static
    address 10.10.10.12/24
    bridge-ports INTERFACE_INTERNAL
    bridge-stp off
    bridge-fd 0

source /etc/network/interfaces.d/*
```

Remplacer les noms génériques par les interfaces réelles de PVE2.

Appliquer :

```bash
ifreload -a
```

Vérifier :

```bash
ip -br addr
```

Résultat attendu :

- `vmbr0` : `192.168.56.12/24`
- `vmbr1` : `10.10.10.12/24`

## 7. Configuration réseau de STORAGE-VM

**Machine concernée : STORAGE-VM (Ubuntu Server)**

Cette machine utilise Ubuntu Server et Netplan pour gérer ses interfaces réseau.

### 7.1 Identifier les interfaces

```bash
ip -br addr
```

Identifier les interfaces Host-only et Internal Network.

### 7.2 Identifier le fichier Netplan

```bash
ls /etc/netplan/
```

Ouvrir le fichier YAML affiché :

```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

Le nom du fichier peut être différent selon l'installation.

### 7.3 Configurer les adresses IP

Exemple de configuration, en remplaçant les noms d'interfaces par ceux réellement présents sur la machine :

```yaml
network:
  version: 2
  ethernets:
    INTERFACE_HOSTONLY:
      dhcp4: false
      addresses:
        - 192.168.56.20/24

    INTERFACE_INTERNAL:
      dhcp4: false
      addresses:
        - 10.10.10.20/24
```

Cette configuration ne définit pas de passerelle par défaut. Les deux réseaux du laboratoire sont locaux.

### 7.4 Appliquer la configuration

```bash
sudo netplan generate
sudo netplan try
```

Si la configuration fonctionne, confirmer son application.

Puis vérifier :

```bash
ip -br addr
```

Résultat attendu :

- Interface Host-only : `192.168.56.20/24`
- Interface Internal Network : `10.10.10.20/24`

## 8. Configuration des noms des machines

Pour faciliter la communication entre les machines, nous allons ajouter les noms des trois serveurs dans le fichier `/etc/hosts`.

Sur PVE1, PVE2 et STORAGE-VM, ouvrir :

```bash
nano /etc/hosts
```

Ajouter les lignes suivantes :

```text
192.168.56.11 pve1.proxmox.lab pve1
192.168.56.12 pve2.proxmox.lab pve2
192.168.56.20 storage.proxmox.lab storage

10.10.10.11 pve1-int
10.10.10.12 pve2-int
10.10.10.20 storage-int
```

Enregistrer le fichier.

Vérifier la résolution :

```bash
getent hosts pve1.proxmox.lab
getent hosts pve2.proxmox.lab
getent hosts storage.proxmox.lab
```

Les noms principaux correspondent aux adresses de gestion. Les alias `-int` servent à identifier les adresses du réseau interne.

## 9. Tests de connectivité

### 9.1 Depuis PVE1

Tester le réseau de gestion :

```bash
ping -c 4 192.168.56.12
ping -c 4 192.168.56.20
```

Tester le réseau interne :

```bash
ping -c 4 10.10.10.12
ping -c 4 10.10.10.20
```

### 9.2 Depuis PVE2

```bash
ping -c 4 192.168.56.11
ping -c 4 192.168.56.20

ping -c 4 10.10.10.11
ping -c 4 10.10.10.20
```

### 9.3 Depuis STORAGE-VM

```bash
ping -c 4 192.168.56.11
ping -c 4 192.168.56.12

ping -c 4 10.10.10.11
ping -c 4 10.10.10.12
```

### 9.4 Depuis Windows 11

Ouvrir PowerShell :

```powershell
ping 192.168.56.11
ping 192.168.56.12
ping 192.168.56.20
```

Les adresses `10.10.10.x` ne sont pas directement accessibles depuis Windows, car elles appartiennent au réseau Internal Network de VirtualBox.

## 10. Configuration réseau des machines invitées Proxmox

Les machines Alpine Linux et Ubuntu Server sont installées à l'intérieur de Proxmox. Elles ne sont pas des machines VirtualBox indépendantes.

Leur connexion réseau passe donc par les bridges Proxmox.

### 10.1 Vérifier les bridges

Dans l'interface Web de Proxmox :

1. Sélectionner PVE1.
2. Ouvrir System → Network.
3. Vérifier la présence de `vmbr0` et `vmbr1`.
4. Répéter sur PVE2.

### 10.2 Connecter Alpine Linux

Dans l'interface Web de Proxmox :

1. Sélectionner la VM Alpine Linux.
2. Aller dans Hardware.
3. Sélectionner Network Device.
4. Cliquer sur Edit.
5. Choisir `vmbr1` comme Bridge.

La VM Alpine sera ainsi connectée au réseau interne.

Exemple d'adresse IP à utiliser si elle est disponible :

```text
10.10.10.101/24
```

### 10.3 Connecter Ubuntu Server

Répéter la même procédure pour la VM Ubuntu Server.

Choisir également `vmbr1` comme Bridge.

Exemple d'adresse IP :

```text
10.10.10.102/24
```

Ces adresses sont des exemples à réserver après vérification qu'elles ne sont pas déjà utilisées.

### 10.4 Vérifier la communication entre les machines invitées

Depuis Alpine Linux :

```bash
ping -c 4 10.10.10.102
ping -c 4 10.10.10.20
```

Depuis Ubuntu Server :

```bash
ping -c 4 10.10.10.101
ping -c 4 10.10.10.20
```

Si les tests réussissent, les machines invitées peuvent communiquer avec le serveur de stockage sur le réseau interne.

Remarque : avec `vmbr1`, les machines invitées ne sont pas directement accessibles depuis Windows 11. Pour les administrer depuis Windows, il faudra prévoir un accès adapté, par exemple via le réseau de gestion ou une configuration de routage supplémentaire.

## 11. Vérification de l'accès aux interfaces Proxmox

Depuis le navigateur de Windows 11, ouvrir :

PVE1 :

```text
https://192.168.56.11:8006
```

PVE2 :

```text
https://192.168.56.12:8006
```

Une alerte de certificat auto-signé peut apparaître. Elle est attendue dans ce laboratoire.

## 12. Dépannage

### Problème 1 : une machine ne répond pas au ping

Vérifier :

```bash
ip -br addr
ip route
```

Vérifier également les paramètres des adaptateurs dans VirtualBox.

### Problème 2 : une interface ne possède pas d'adresse IP

Vérifier le fichier de configuration correspondant :

```bash
cat /etc/network/interfaces
```

Pour Ubuntu :

```bash
sudo netplan get
```

### Problème 3 : perte de connexion après modification

Utiliser la console VirtualBox pour accéder directement à la machine.

Sur Proxmox, restaurer la sauvegarde si nécessaire :

```bash
cp /etc/network/interfaces.backup /etc/network/interfaces
ifreload -a
```

Sur Ubuntu, corriger le fichier Netplan puis appliquer :

```bash
sudo netplan generate
sudo netplan try
```

### Problème 4 : les machines invitées ne communiquent pas

Vérifier :

- Que les deux machines utilisent le bon bridge.
- Que les adresses IP appartiennent au même sous-réseau.
- Que les adresses IP ne sont pas dupliquées.
- Que le pare-feu ne bloque pas les tests ICMP.

## 13. Résultats attendus

À la fin de cette étape :

- PVE1 et PVE2 communiquent sur le réseau Host-only.
- PVE1 et PVE2 communiquent sur le réseau Internal Network.
- STORAGE-VM communique avec les deux nœuds Proxmox.
- Windows 11 accède aux interfaces Web des deux nœuds.
- Alpine Linux et Ubuntu Server disposent d'une connexion au réseau interne Proxmox.
- Les machines invitées peuvent communiquer avec STORAGE-VM sur le réseau interne.

## 14. Conclusion

La configuration réseau du laboratoire est maintenant préparée. Les deux nœuds Proxmox disposent d'un réseau de gestion et d'un réseau interne commun. Le serveur STORAGE-VM est connecté aux deux réseaux et les machines invitées peuvent utiliser le bridge interne.

Cette infrastructure constitue la base nécessaire pour les prochaines étapes : création du cluster Proxmox, configuration du stockage NFS, migration des machines virtuelles et mise en place du QDevice.
