# 01 – Prérequis et budget des ressources

## 1. Objectif

Avant de commencer la création de notre infrastructure Proxmox VE, il est nécessaire de vérifier que l'ordinateur dispose des ressources matérielles et logicielles suffisantes.

Ce projet utilise la virtualisation imbriquée : Oracle VirtualBox héberge deux machines virtuelles Proxmox VE, qui hébergeront à leur tour des machines virtuelles Linux.

Cette première étape permet de :
- Vérifier la compatibilité matérielle de l'ordinateur.
- Préparer l'environnement Windows 11.
- Identifier les logiciels nécessaires.
- Définir la répartition des ressources.
- Prévenir les problèmes liés à la virtualisation imbriquée.

## 2. Environnement du projet

L'infrastructure repose sur les composants suivants :

| Composant | Configuration |
|---|---|
| Système hôte | Windows 11 |
| Mémoire physique | 16 Go RAM |
| Hyperviseur externe | Oracle VirtualBox |
| Premier hyperviseur | Proxmox VE (PVE1) |
| Deuxième hyperviseur | Proxmox VE (PVE2) |
| VM invitée sur PVE1 | Alpine Linux |
| VM invitée sur PVE2 | Ubuntu Server |
| Serveur supplémentaire | Ubuntu Server |
| Services supplémentaires | NFS et Corosync QNetd |

## 3. Prérequis matériels

L'ordinateur doit disposer des caractéristiques suivantes :

| Ressource | Minimum / recommandation |
|---|---|
| Système | Windows 11 64 bits |
| RAM | 16 Go recommandés pour ce laboratoire |
| Processeur | Intel ou AMD compatible avec la virtualisation matérielle |
| Virtualisation | Intel VT-x ou AMD-V activée |
| Disque | Espace libre suffisant pour les ISO et les machines virtuelles |
| Réseau | Connexion Internet et interfaces réseau virtuelles |

**Remarque :** 16 Go de RAM représentent une configuration de laboratoire relativement limitée pour faire fonctionner simultanément trois machines virtuelles VirtualBox et plusieurs machines invitées Proxmox. Il faudra limiter les ressources des machines et éviter de démarrer toutes les charges de travail simultanément si les performances deviennent insuffisantes.

## 4. Prérequis logiciels

Les logiciels et systèmes suivants sont nécessaires :

| Logiciel / système | Utilisation |
|---|---|
| Windows 11 | Système d'exploitation principal |
| Oracle VirtualBox | Hébergement des machines Proxmox et du serveur supplémentaire |
| Proxmox VE ISO | Installation des deux hyperviseurs |
| Alpine Linux ISO | Installation de la VM invitée sur PVE1 |
| Ubuntu Server ISO | Installation de la VM invitée sur PVE2 |
| Ubuntu Server ISO | Installation du serveur NFS/QDevice |

Les images ISO doivent être téléchargées depuis leurs sites officiels.

- VirtualBox : https://www.virtualbox.org/
- Proxmox VE : https://www.proxmox.com/
- Alpine Linux : https://alpinelinux.org/
- Ubuntu Server : https://ubuntu.com/download/server

## 5. Vérification de la virtualisation matérielle

La virtualisation matérielle doit être activée dans le BIOS/UEFI de l'ordinateur.

### Étape 1 : Vérifier la virtualisation sous Windows

**Emplacement : Windows 11**

1. Ouvrir le Gestionnaire des tâches avec `Ctrl + Shift + Esc`.
2. Accéder à l'onglet **Performances**.
3. Sélectionner **Processeur**.
4. Repérer la ligne **Virtualisation**.

Résultat attendu :

```text
Virtualisation : Activée
```

Si la virtualisation est désactivée, il faudra l'activer dans le BIOS/UEFI.

### Étape 2 : Vérifier la configuration de virtualisation

**Emplacement : Windows 11 – PowerShell**

Exécuter :

```powershell
systeminfo
```

Consulter les informations relatives à la virtualisation Hyper-V.

Selon la configuration de Windows, certaines lignes peuvent indiquer qu'un hyperviseur est déjà détecté. Cela ne signifie pas automatiquement que VirtualBox ne fonctionnera pas, mais il faudra vérifier les performances et la prise en charge de la virtualisation imbriquée.

## 6. Budget mémoire RAM

L'ordinateur possède 16 Go de mémoire RAM.

La répartition initiale proposée est la suivante :

| Machine | RAM attribuée |
|---|---:|
| Windows 11 | 4 Go réservés |
| PVE1 | 4 Go |
| PVE2 | 4 Go |
| STORAGE-VM | 1 Go |
| Mémoire restante | 3 Go |
| **Total** | **16 Go** |

### Explication

- Windows 11 utilise une partie de la mémoire pour le système et les applications.
- PVE1 utilise ses 4 Go pour Proxmox VE et sa VM Alpine Linux.
- PVE2 utilise ses 4 Go pour Proxmox VE et sa VM Ubuntu Server.
- STORAGE-VM utilise 1 Go pour Ubuntu Server, NFS et QNetd.
- Les 3 Go restants constituent une marge pour les besoins du système et les variations de consommation.

**Attention :** les ressources des VM invitées sont prélevées sur la RAM attribuée à leur hyperviseur Proxmox. Elles ne doivent pas être ajoutées une deuxième fois au budget global.

## 7. Préparation de la virtualisation imbriquée

La virtualisation imbriquée permet d'exécuter un hyperviseur à l'intérieur d'une machine virtuelle.

Dans notre projet :

```text
Windows 11
    |
    Oracle VirtualBox
        |
        Proxmox VE
            |
            Machines virtuelles Linux
```

Cette architecture nécessite que les fonctionnalités de virtualisation du processeur soient accessibles aux machines Proxmox.

### Configuration dans VirtualBox

Pour PVE1 et PVE2 :

1. Arrêter complètement la machine virtuelle.
2. Ouvrir VirtualBox.
3. Sélectionner la machine virtuelle Proxmox.
4. Ouvrir **Configuration**.
5. Accéder à **Système → Processeur**.
6. Activer l'option de virtualisation imbriquée si elle est disponible.
7. Vérifier que le nombre de processeurs attribués est adapté à l'ordinateur.

Répéter l'opération pour les deux machines Proxmox.

### Problèmes possibles

Certaines fonctionnalités Windows peuvent affecter les performances de VirtualBox :

- Hyper-V.
- Windows Hypervisor Platform.
- Virtual Machine Platform.
- WSL2.
- Isolation du noyau et intégrité de la mémoire (Core Isolation / Memory Integrity).

Il ne faut pas désactiver automatiquement ces fonctionnalités. En cas de problème, identifier d'abord le conflit et consulter la documentation VirtualBox avant de modifier les paramètres de sécurité de Windows.

## 8. Vérification de la disponibilité de KVM

Après l'installation de Proxmox VE, il faudra vérifier que la virtualisation imbriquée fonctionne correctement.

**Emplacement : PVE1 et PVE2**

Exécuter :

```bash
egrep -c '(vmx|svm)' /proc/cpuinfo
```

Un résultat supérieur à zéro indique que les extensions de virtualisation du processeur sont visibles.

Vérifier également :

```bash
ls -l /dev/kvm
```

Résultat attendu : le périphérique `/dev/kvm` doit exister.

Si KVM n'est pas disponible, les machines virtuelles invitées peuvent ne pas démarrer ou fonctionner avec des performances fortement dégradées.

## 9. Vérification finale des prérequis

Avant de passer à l'installation, vérifier les éléments suivants :

- [ ] Windows 11 est opérationnel.
- [ ] VirtualBox est installé.
- [ ] La virtualisation matérielle est activée.
- [ ] Les images ISO nécessaires sont disponibles.
- [ ] L'espace disque est suffisant.
- [ ] La répartition de la RAM est définie.
- [ ] La virtualisation imbriquée est prévue pour PVE1 et PVE2.
- [ ] Les éventuels conflits avec les fonctionnalités Windows ont été identifiés.

## 10. Résultat attendu

À la fin de cette étape, l'environnement doit être prêt pour la création des trois machines virtuelles VirtualBox :

- PVE1 : premier hyperviseur Proxmox VE.
- PVE2 : deuxième hyperviseur Proxmox VE.
- STORAGE-VM : serveur Ubuntu destiné au stockage NFS et au service QNetd.

## 11. Conclusion

Cette étape permet de préparer les ressources matérielles et logicielles nécessaires à la réalisation du projet.

La vérification de la virtualisation matérielle et la planification de la mémoire sont particulièrement importantes, car l'infrastructure repose sur la virtualisation imbriquée.

La prochaine étape consiste à installer et configurer Oracle VirtualBox sous Windows 11.
