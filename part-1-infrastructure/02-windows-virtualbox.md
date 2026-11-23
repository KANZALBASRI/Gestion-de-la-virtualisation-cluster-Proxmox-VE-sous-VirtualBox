# 02 – Installation et configuration de VirtualBox sous Windows 11

## 1. Objectif

Après avoir vérifié les prérequis matériels et logiciels, cette étape consiste à installer et configurer Oracle VirtualBox sur Windows 11.

VirtualBox sera utilisé comme hyperviseur externe pour héberger trois machines virtuelles :

- **PVE1 :** premier serveur Proxmox VE.
- **PVE2 :** deuxième serveur Proxmox VE.
- **STORAGE-VM :** serveur Ubuntu destiné au stockage NFS et au service Corosync QNetd.

Les deux serveurs Proxmox hébergeront ensuite leurs propres machines virtuelles Linux.

Cette étape permet de :
- Installer Oracle VirtualBox.
- Préparer les réseaux virtuels.
- Créer les trois machines virtuelles.
- Configurer les ressources matérielles.
- Activer la virtualisation imbriquée.
- Préparer l'environnement pour l'installation de Proxmox VE.

## 2. Téléchargement des logiciels nécessaires

Avant de commencer, télécharger les logiciels suivants depuis leurs sites officiels.

| Logiciel | Utilisation | Site officiel |
|---|---|---|
| Oracle VirtualBox | Hyperviseur externe | https://www.virtualbox.org/wiki/Downloads |
| Proxmox VE ISO | Installation de PVE1 et PVE2 | https://www.proxmox.com/en/downloads |
| Ubuntu Server ISO | Installation de STORAGE-VM | https://ubuntu.com/download/server |

Les images ISO d'Alpine Linux et d'Ubuntu Server destinées aux machines invitées Proxmox seront utilisées dans une étape ultérieure.

**Remarque :** VirtualBox Extension Pack n'est pas nécessaire pour réaliser les configurations réseau et les installations décrites dans ce guide.

## 3. Installation d'Oracle VirtualBox

### Étape 1 : Télécharger VirtualBox

**Emplacement : Windows 11**

1. Ouvrir le site officiel :
   https://www.virtualbox.org/wiki/Downloads
2. Télécharger la version correspondant à Windows.
3. Attendre la fin du téléchargement.

### Étape 2 : Installer VirtualBox

1. Ouvrir le fichier d'installation téléchargé.
2. Cliquer sur Next.
3. Conserver les composants proposés par défaut.
4. Choisir le dossier d'installation.
5. Accepter l'installation des interfaces réseau VirtualBox.
6. Cliquer sur Install.
7. Autoriser les modifications demandées par Windows.
8. Terminer l'installation.

Pendant l'installation, Windows peut temporairement interrompre la connexion réseau lors de la configuration des interfaces virtuelles.

### Étape 3 : Vérifier l'installation

Lancer Oracle VirtualBox depuis le menu Démarrer.

Résultat attendu :

- L'interface principale de VirtualBox apparaît.
- Le gestionnaire de machines virtuelles est accessible.
- La création de nouvelles machines virtuelles est disponible.

**Capture d'écran :**

`part-1-infrastructure/screenshots/03-virtualbox-installed.png`

Légende : Interface principale d'Oracle VirtualBox après installation.

## 4. Configuration des réseaux virtuels

Notre infrastructure utilise deux réseaux distincts afin de séparer l'accès de gestion et la communication interne.

| Réseau | Type VirtualBox | Adresse réseau | Utilisation |
|---|---|---|---|
| Réseau de gestion | Host-only | 192.168.56.0/24 | Communication entre Windows et les VM |
| Réseau interne | Internal Network | 10.10.10.0/24 | Communication entre PVE1, PVE2 et STORAGE-VM |

### 4.1. Création du réseau Host-only

Le réseau Host-only permet à Windows de communiquer directement avec les machines virtuelles sans nécessiter une connexion Internet.

**Emplacement : VirtualBox**

1. Ouvrir VirtualBox.
2. Accéder à Tools → Network.
3. Ouvrir l'onglet Host-only Networks.
4. Cliquer sur Create.
5. Sélectionner le réseau créé.
6. Configurer l'adresse IPv4.

Paramètres :

| Paramètre | Valeur |
|---|---|
| Adresse IPv4 | 192.168.56.1 |
| Masque | 255.255.255.0 |
| DHCP | Désactivé |

Le serveur DHCP peut être désactivé puisque nous utiliserons des adresses IP statiques dans notre infrastructure.

Résultat attendu : un réseau Host-only est disponible avec l'adresse `192.168.56.1/24`.

**Capture d'écran :**

`part-1-infrastructure/screenshots/04-host-only-network.png`

### 4.2. Création du réseau Internal Network

Le réseau Internal Network permet aux machines virtuelles de communiquer entre elles sans exposer directement ce réseau à Windows.

Contrairement au réseau Host-only, il ne nécessite pas la création d'une interface réseau sur l'hôte Windows.

**Emplacement : VirtualBox**

1. Sélectionner une machine virtuelle.
2. Ouvrir Configuration → Réseau.
3. Activer l'Adaptateur 2.
4. Dans Mode d'accès réseau, sélectionner Internal Network.
5. Donner le nom suivant au réseau :

`intnet-proxmox`

6. Répéter cette configuration sur les trois machines virtuelles.

**Important :** le nom du réseau interne doit être exactement identique sur PVE1, PVE2 et STORAGE-VM.

Résultat attendu : les trois machines pourront communiquer sur le même réseau interne.

## 5. Création de la machine virtuelle PVE1

PVE1 sera le premier hyperviseur Proxmox de notre infrastructure.

### 5.1. Création de la machine

**Emplacement : VirtualBox**

1. Cliquer sur New.
2. Nommer la machine `PVE1`.
3. Sélectionner Linux comme type.
4. Choisir Debian (64-bit) si cette option est proposée.
5. Sélectionner l'image ISO de Proxmox VE.

### 5.2. Configuration matérielle

Attribuer les ressources suivantes :

| Paramètre | Valeur |
|---|---|
| Nom | PVE1 |
| Système | Proxmox VE |
| Mémoire RAM | 4096 Mo |
| Processeurs | 2 vCPU |
| Disque virtuel | 50 Go |
| Type de disque | VDI |
| Allocation | Dynamique |

Le disque à allocation dynamique permet de limiter l'espace physique utilisé tant que le disque virtuel n'est pas entièrement rempli.

**Remarque :** le disque doit disposer d'un espace suffisant pour Proxmox et les machines virtuelles hébergées.

### 5.3. Configuration réseau

Accéder à Configuration → Réseau.

**Adaptateur 1 :**

| Paramètre | Valeur |
|---|---|
| Activer l'adaptateur | Oui |
| Mode d'accès réseau | Host-only Adapter |
| Nom | Réseau Host-only créé précédemment |
| Mode promiscuité | Allow All |

**Adaptateur 2 :**

| Paramètre | Valeur |
|---|---|
| Activer l'adaptateur | Oui |
| Mode d'accès réseau | Internal Network |
| Nom | intnet-proxmox |
| Mode promiscuité | Allow All |

Le mode promiscuité Allow All est important pour permettre aux machines invitées de Proxmox d'utiliser correctement les interfaces réseau pontées lorsque cela est nécessaire.

### 5.4. Vérification

Avant de démarrer PVE1, vérifier :

- [ ] La mémoire RAM est de 4 Go.
- [ ] Le processeur dispose de 2 vCPU.
- [ ] Le disque virtuel est de 50 Go.
- [ ] L'adaptateur Host-only est activé.
- [ ] L'adaptateur Internal Network est activé.
- [ ] La virtualisation imbriquée sera activée.

**Capture d'écran :**

`part-1-infrastructure/screenshots/05-pve1-vm-configuration.png`

## 6. Création de la machine virtuelle PVE2

PVE2 sera le deuxième hyperviseur Proxmox.

La configuration sera identique à celle de PVE1 afin de faciliter la création du cluster.

### 6.1. Création de la machine

1. Cliquer sur New.
2. Nommer la machine `PVE2`.
3. Sélectionner Linux.
4. Choisir Debian (64-bit) si disponible.
5. Sélectionner la même image ISO Proxmox VE.

### 6.2. Configuration matérielle

| Paramètre | Valeur |
|---|---|
| Nom | PVE2 |
| Système | Proxmox VE |
| Mémoire RAM | 4096 Mo |
| Processeurs | 2 vCPU |
| Disque virtuel | 50 Go |
| Type de disque | VDI |
| Allocation | Dynamique |

### 6.3. Configuration réseau

Configurer les deux adaptateurs comme pour PVE1 :

- Adaptateur 1 : Host-only Adapter.
- Adaptateur 2 : Internal Network.
- Nom du réseau interne : `intnet-proxmox`.
- Mode promiscuité : Allow All.

### 6.4. Vérification

- [ ] RAM de 4 Go.
- [ ] 2 vCPU.
- [ ] Disque de 50 Go.
- [ ] Réseau Host-only activé.
- [ ] Réseau interne activé.

**Capture d'écran :**

`part-1-infrastructure/screenshots/06-pve2-vm-configuration.png`

## 7. Création de la machine STORAGE-VM

STORAGE-VM est une machine virtuelle supplémentaire qui hébergera deux services :

- Serveur NFS pour le stockage partagé.
- Corosync QNetd pour le QDevice du cluster.

Cette machine ne sera pas un nœud du cluster Proxmox.

### 7.1. Création de la machine

1. Cliquer sur New.
2. Nommer la machine `STORAGE-VM`.
3. Sélectionner Linux.
4. Choisir Ubuntu (64-bit) si disponible.
5. Sélectionner l'image ISO Ubuntu Server.

### 7.2. Configuration matérielle

| Paramètre | Valeur |
|---|---|
| Nom | STORAGE-VM |
| Système | Ubuntu Server |
| Mémoire RAM | 1024 Mo |
| Processeurs | 1 vCPU |
| Disque virtuel | 20 Go |
| Type de disque | VDI |
| Allocation | Dynamique |

### 7.3. Configuration réseau

Configurer :

**Adaptateur 1 :**
- Mode : Host-only Adapter.
- Réseau : réseau de gestion.
- Mode promiscuité : Allow All.

**Adaptateur 2 :**
- Mode : Internal Network.
- Nom : `intnet-proxmox`.
- Mode promiscuité : Allow All.

### 7.4. Vérification

- [ ] RAM de 1 Go.
- [ ] 1 vCPU.
- [ ] Disque de 20 Go.
- [ ] Adaptateur Host-only activé.
- [ ] Adaptateur Internal Network activé.

**Capture d'écran :**

`part-1-infrastructure/screenshots/07-storage-vm-configuration.png`

## 8. Activation de la virtualisation imbriquée

La virtualisation imbriquée est indispensable pour permettre à Proxmox VE d'exécuter ses propres machines virtuelles.

Dans notre architecture :

```text
Windows 11
    |
    Oracle VirtualBox
        |
        +-- PVE1 (Proxmox VE)
        |      |
        |      +-- Alpine Linux
        |
        +-- PVE2 (Proxmox VE)
        |      |
        |      +-- Ubuntu Server
        |
        +-- STORAGE-VM (Ubuntu Server)
               |
               +-- NFS
               +-- Corosync QNetd
```

### 8.1. Activation depuis l'interface VirtualBox

Pour chaque machine PVE :

1. Arrêter complètement la machine virtuelle.
2. Sélectionner PVE1.
3. Ouvrir Configuration → Système → Processeur.
4. Rechercher l'option Enable Nested VT-x/AMD-V, si elle est disponible.
5. Activer cette option.
6. Répéter l'opération pour PVE2.

Cette option ne concerne pas STORAGE-VM.

### 8.2. Activation depuis PowerShell

Si l'option n'est pas disponible dans l'interface graphique, il est possible de vérifier la configuration avec VBoxManage.

**Emplacement : Windows 11 – PowerShell**

Vérifier les machines disponibles :

```powershell
& "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" list vms
```

Activer la virtualisation imbriquée sur PVE1 :

```powershell
& "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" modifyvm "PVE1" --nested-hw-virt on
```

Activer la virtualisation imbriquée sur PVE2 :

```powershell
& "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" modifyvm "PVE2" --nested-hw-virt on
```

Vérifier la configuration :

```powershell
& "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" showvminfo "PVE1"
```

Répéter pour PVE2.

**Remarque :** la prise en charge de la virtualisation imbriquée dépend de la version de VirtualBox, du processeur et de la configuration de l'hyperviseur Windows. Si la commande échoue, consulter la documentation officielle avant de poursuivre.

**Capture d'écran :**

`part-1-infrastructure/screenshots/08-nested-virtualization.png`

## 9. Vérification de la configuration générale

Avant de passer à l'installation de Proxmox VE, vérifier que les trois machines virtuelles sont correctement configurées.

### 9.1. Tableau récapitulatif

| Paramètre | PVE1 | PVE2 | STORAGE-VM |
|---|---|---|---|
| Système | Proxmox VE | Proxmox VE | Ubuntu Server |
| RAM | 4 Go | 4 Go | 1 Go |
| CPU | 2 vCPU | 2 vCPU | 1 vCPU |
| Disque | 50 Go | 50 Go | 20 Go |
| Adaptateur 1 | Host-only | Host-only | Host-only |
| Adaptateur 2 | Internal | Internal | Internal |
| Virtualisation imbriquée | Activée | Activée | Non nécessaire |

### 9.2. Budget des ressources

La mémoire attribuée aux trois machines représente :

| Machine | RAM |
|---|---:|
| PVE1 | 4 Go |
| PVE2 | 4 Go |
| STORAGE-VM | 1 Go |
| **Total attribué** | **9 Go** |

Avec 16 Go de RAM physiques, environ 7 Go restent disponibles pour Windows et les autres applications lorsque les trois machines sont démarrées.

La mémoire des machines invitées Proxmox est comprise dans les 4 Go attribués à chaque nœud. Il faut donc éviter de surcharger les hyperviseurs.

### 9.3. Vérification des réseaux

Vérifier que les trois machines utilisent :

- Le même réseau Host-only pour la gestion.
- Le même réseau Internal Network nommé `intnet-proxmox`.

Les adresses IP statiques seront configurées dans le prochain guide.

## 10. Dépannage

### Problème 1 : VirtualBox ne démarre pas correctement

Causes possibles :
- Conflit avec Hyper-V.
- Problème de pilote VirtualBox.
- Fonctionnalités de virtualisation Windows incompatibles avec la configuration utilisée.

Solution :
1. Vérifier que VirtualBox est correctement installé.
2. Vérifier les messages d'erreur.
3. Consulter la documentation VirtualBox.
4. Identifier le conflit avant de modifier les fonctionnalités Windows.

Ne pas désactiver automatiquement Hyper-V, WSL2 ou les protections de sécurité Windows.

### Problème 2 : Virtualisation imbriquée indisponible

Causes possibles :
- VT-x ou AMD-V désactivé dans le BIOS/UEFI.
- Processeur ou version VirtualBox ne prenant pas en charge la configuration demandée.
- Hyperviseur Windows utilisant les extensions matérielles.

Solution :
1. Vérifier la virtualisation dans le Gestionnaire des tâches.
2. Vérifier la version de VirtualBox.
3. Vérifier la configuration des machines PVE.
4. Consulter les messages d'erreur de VBoxManage.

### Problème 3 : Les machines ne communiquent pas

Causes possibles :
- Réseaux virtuels différents.
- Nom incorrect du réseau interne.
- Adaptateur désactivé.
- Configuration IP incorrecte.

Solution :
1. Vérifier les deux adaptateurs réseau.
2. Vérifier que le nom `intnet-proxmox` est identique.
3. Vérifier les adresses IP.
4. Tester la connectivité après l'installation des systèmes.

### Problème 4 : Mémoire insuffisante

Causes possibles :
- Trop de machines démarrées simultanément.
- Applications Windows consommant trop de RAM.
- Ressources attribuées trop importantes.

Solution :
1. Fermer les applications inutilisées.
2. Démarrer uniquement les machines nécessaires.
3. Réduire temporairement la mémoire des machines si cela reste compatible avec leurs services.

## 11. Vérification finale

Avant de continuer, vérifier les éléments suivants :

- [ ] Oracle VirtualBox est installé.
- [ ] Le réseau Host-only est créé.
- [ ] Le réseau Internal Network est créé.
- [ ] PVE1 est créé.
- [ ] PVE2 est créé.
- [ ] STORAGE-VM est créée.
- [ ] Les ressources CPU et RAM sont configurées.
- [ ] Les deux adaptateurs réseau sont configurés sur chaque machine.
- [ ] La virtualisation imbriquée est activée sur PVE1.
- [ ] La virtualisation imbriquée est activée sur PVE2.
- [ ] Les images ISO sont disponibles.

## 12. Résultat attendu

À la fin de cette étape, l'environnement VirtualBox doit contenir trois machines virtuelles prêtes à être utilisées :

- PVE1 : premier hyperviseur Proxmox VE.
- PVE2 : deuxième hyperviseur Proxmox VE.
- STORAGE-VM : futur serveur NFS et QNetd.

Les réseaux Host-only et Internal Network sont également préparés.

Aucune installation de Proxmox VE ou d'Ubuntu Server n'est encore nécessairement terminée à ce stade.

## 13. Conclusion

Cette étape a permis de préparer Oracle VirtualBox et de créer l'environnement virtuel nécessaire au projet.

La configuration des réseaux et l'activation de la virtualisation imbriquée constituent des éléments essentiels pour assurer le bon fonctionnement des deux hyperviseurs Proxmox.

La prochaine étape consiste à installer Proxmox VE sur PVE1 et PVE2, puis à installer Ubuntu Server sur STORAGE-VM.
