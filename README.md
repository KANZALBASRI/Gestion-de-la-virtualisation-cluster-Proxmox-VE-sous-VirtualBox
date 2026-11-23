# Gestion de la virtualisation : cluster Proxmox VE sous VirtualBox

Guide pratique, pas à pas, pour construire un **cluster Proxmox VE à 2 nœuds** dans VirtualBox sur Windows 11, avec stockage partagé NFS, migration de machines virtuelles, QDevice et haute disponibilité (HA).

Ce dépôt s'adresse aux étudiants qui découvrent Proxmox VE. Chaque étape explique ce que l'on réalise, pourquoi on le réalise, les commandes à exécuter, les résultats attendus et les solutions aux problèmes courants.

## 1. Objectifs du projet

- Déployer une infrastructure de virtualisation imbriquée (VirtualBox comme hyperviseur de type 2 et Proxmox VE comme hyperviseur de type 1).
- Mettre en place un cluster Proxmox VE composé de deux nœuds.
- Configurer un stockage partagé via NFS entre les deux nœuds.
- Migrer des machines virtuelles d'un nœud vers l'autre.
- Ajouter un QDevice pour améliorer le maintien du quorum.
- Configurer la haute disponibilité (HA) et simuler une panne d'un nœud.
- Vérifier le fonctionnement global de l'infrastructure.

## 2. Architecture

```text
Windows 11 (hôte physique - 16 Go RAM)
│
└── Oracle VirtualBox
    │
    ├── PVE1 : Proxmox VE
    │   └── VM invitée : Alpine Linux
    │
    ├── PVE2 : Proxmox VE
    │   └── VM invitée : Ubuntu Server
    │
    └── STORAGE-VM : Ubuntu Server
        ├── Serveur NFS
        └── Corosync QNetd (QDevice)
```

Les deux nœuds Proxmox constituent le cluster. La troisième machine virtuelle héberge les services NFS et QNetd, sans être elle-même un nœud du cluster.

Les schémas détaillés sont disponibles dans le dossier `diagrams/`.

## 3. Machines et réseau

Les valeurs suivantes sont des **exemples de configuration**. Elles doivent être adaptées à l'environnement VirtualBox et conservées de manière cohérente dans tous les guides.

| Machine | Rôle | Système | RAM attribuée | IP gestion | IP réseau interne |
|---|---|---|---:|---|---|
| Hôte | Ordinateur physique | Windows 11 | 4 Go réservés | 192.168.56.1 | - |
| PVE1 | Nœud Proxmox 1 | Proxmox VE | 4 Go | 192.168.56.11 | 10.10.10.11 |
| PVE2 | Nœud Proxmox 2 | Proxmox VE | 4 Go | 192.168.56.12 | 10.10.10.12 |
| STORAGE-VM | NFS + QNetd | Ubuntu Server | 1 Go | 192.168.56.20 | 10.10.10.20 |
| VM invitée 1 | Hébergée sur PVE1 | Alpine Linux | 512 Mo à 1 Go | Selon configuration | Selon configuration |
| VM invitée 2 | Hébergée sur PVE2 | Ubuntu Server | 1 à 2 Go | Selon configuration | Selon configuration |

### Organisation du réseau VirtualBox

Deux interfaces réseau sont prévues pour chaque machine virtuelle :

- **Adaptateur 1 : Host-only** — communication de gestion entre Windows et les machines virtuelles.
- **Adaptateur 2 : Internal Network** — communication entre les deux nœuds Proxmox et le serveur NFS/QDevice.

Le réseau de gestion utilise l'exemple `192.168.56.0/24`.

Le réseau interne utilise l'exemple `10.10.10.0/24`.

Le serveur NFS sera accessible depuis les nœuds Proxmox via son adresse interne `10.10.10.20`.

La configuration des interfaces réseau et des bridges Proxmox sera détaillée dans le guide correspondant.

## 4. Budget mémoire RAM

L'ordinateur hôte dispose de 16 Go de RAM.

| Composant | RAM prévisionnelle |
|---|---:|
| Windows 11 | 4 Go |
| PVE1 | 4 Go |
| PVE2 | 4 Go |
| STORAGE-VM | 1 Go |
| Marge disponible | 3 Go |
| **Total** | **16 Go** |

**Important :** la mémoire des machines virtuelles Alpine Linux et Ubuntu Server est comprise dans les ressources attribuées à PVE1 et PVE2. Elle ne doit pas être comptabilisée une deuxième fois.

Cette répartition est indicative et doit être adaptée en fonction de la consommation réelle des systèmes.

## 5. Prérequis

### Matériel

- Ordinateur sous Windows 11.
- 16 Go de RAM minimum recommandés pour cette configuration de laboratoire.
- Processeur compatible avec la virtualisation matérielle Intel VT-x ou AMD-V.
- Virtualisation activée dans le BIOS/UEFI.
- Espace disque suffisant pour les trois machines virtuelles et les machines invitées.

### Logiciels

- Oracle VirtualBox.
- Image ISO de Proxmox VE.
- Image ISO d'Alpine Linux.
- Image ISO d'Ubuntu Server.
- Connexion Internet pour télécharger les paquets nécessaires.

Les images ISO ne sont pas incluses dans ce dépôt.

### Points de vigilance

- La virtualisation imbriquée doit être activée pour PVE1 et PVE2.
- Certaines configurations Hyper-V, WSL2 et Core Isolation peuvent affecter les performances de VirtualBox.
- Il faut vérifier que KVM est disponible dans Proxmox avant de créer les machines virtuelles invitées.

## 6. Plan du guide

### Partie 1 : Infrastructure

| Étape | Guide | Contenu |
|---|---|---|
| 1 | `01-requirements-and-ram-budget.md` | Prérequis matériels, logiciels et RAM |
| 2 | `02-windows-virtualbox.md` | Installation et configuration de VirtualBox |
| 3 | `03-proxmox-installation.md` | Création et installation des deux nœuds Proxmox |
| 4 | `04-network-configuration.md` | Configuration des interfaces et connectivité |
| 5 | `05-cluster-creation.md` | Création du cluster Proxmox |
| 6 | `06-guest-vms.md` | Installation d'Alpine sur PVE1 et Ubuntu sur PVE2 |
| 7 | `07-storage-qdevice-vm.md` | Création et préparation du serveur NFS/QDevice |

### Partie 2 : Travaux pratiques

| TP | Sujet | Guide |
|---|---|---|
| 1 | Stockage partagé NFS | `guide.md` |
| 2 | Migration des machines virtuelles | `guide.md` |
| 3 | QDevice et quorum | `guide.md` |
| 4 | Haute disponibilité | `guide.md` |
| 5 | Tests et validation | `guide.md` |

**Ordre recommandé :** NFS → Migration → QDevice → Haute disponibilité → Tests finaux.

La haute disponibilité nécessite un stockage accessible depuis les deux nœuds et un quorum fonctionnel.

## 7. Structure du dépôt

```text
proxmox-virtualization-management/
│
├── README.md
│
├── part-1-infrastructure/
│   ├── 01-requirements-and-ram-budget.md
│   ├── 02-windows-virtualbox.md
│   ├── 03-proxmox-installation.md
│   ├── 04-network-configuration.md
│   ├── 05-cluster-creation.md
│   ├── 06-guest-vms.md
│   ├── 07-storage-qdevice-vm.md
│   ├── commands.md
│   └── screenshots/
│
├── part-2-practical/
│   ├── 01-nfs-shared-storage/
│   │   ├── guide.md
│   │   ├── commands.md
│   │   └── screenshots/
│   │
│   ├── 02-vm-migration/
│   │   ├── guide.md
│   │   ├── commands.md
│   │   └── screenshots/
│   │
│   ├── 03-qdevice-quorum/
│   │   ├── guide.md
│   │   ├── commands.md
│   │   └── screenshots/
│   │
│   ├── 04-high-availability/
│   │   ├── guide.md
│   │   ├── commands.md
│   │   └── screenshots/
│   │
│   └── 05-testing-and-validation/
│       ├── guide.md
│       ├── commands.md
│       └── screenshots/
│
├── scripts/
│
├── diagrams/
│   ├── infrastructure.png
│   ├── network-topology.png
│   └── storage-architecture.png
│
└── reports/
    └── final-report.pdf
```

## 8. Conventions utilisées dans les guides

Chaque guide doit respecter les conventions suivantes :

- Chaque commande indique l'endroit où elle doit être exécutée : Windows, PVE1, PVE2, STORAGE-VM ou machine virtuelle invitée.
- Chaque guide contient les sections suivantes : objectif, concepts, environnement, prérequis, étapes détaillées, vérification, dépannage, résultats et conclusion.
- Chaque étape explique son utilité avant de présenter les commandes.
- Les résultats attendus sont indiqués après les commandes.
- Les captures d'écran sont numérotées dans l'ordre des étapes : `01-...png`, `02-...png`, etc.
- Les adresses IP et noms des machines doivent rester cohérents dans tous les documents.

## 9. Dépannage général

Les problèmes courants sont documentés dans les guides concernés, notamment :

- Conflits entre VirtualBox et Hyper-V.
- Absence de virtualisation imbriquée.
- Indisponibilité de KVM dans Proxmox.
- Problèmes de connectivité entre les nœuds.
- Échec de création ou de jonction au cluster.
- Problèmes de montage du stockage NFS.
- Perte de quorum.
- Échec de migration ou de redémarrage HA.

## 10. Limitations du projet

Cette infrastructure est un laboratoire pédagogique utilisant la virtualisation imbriquée.

Les deux nœuds Proxmox et le serveur NFS/QDevice sont hébergés sur le même ordinateur physique. Une panne de cet ordinateur entraîne donc l'indisponibilité de toute l'infrastructure.

Le QDevice améliore le fonctionnement du quorum lors de la perte d'un nœud, mais ne garantit pas la disponibilité en cas de panne de l'hôte physique.

Les tests de haute disponibilité portent sur la défaillance simulée d'un nœud Proxmox.

## 12. Auteurs

- Kanza Lbasri
- Hafssa Lahroumi
- Fatima Zahra HMADOUCH

## 13. Encadrant

Mohamed Benaddy
