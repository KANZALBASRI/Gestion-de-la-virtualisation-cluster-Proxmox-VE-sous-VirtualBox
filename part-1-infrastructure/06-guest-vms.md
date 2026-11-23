# 06 – Création et configuration des machines virtuelles invitées

## 1. Objectif

Après la création du cluster Proxmox VE à deux nœuds, cette étape consiste à créer deux machines virtuelles invitées (Guest VMs), installer leurs systèmes d'exploitation et configurer leur environnement.

Pour réduire la consommation des ressources, Alpine Linux remplace Red Hat Enterprise Linux.

### Architecture

| Machine virtuelle | Nœud Proxmox | Système |
|---|---|---|
| Alpine-VM | PVE1 | Alpine Linux |
| Ubuntu-VM | PVE2 | Ubuntu Server |

Infrastructure :
- Hôte : Fedora 44.
- Hyperviseur externe : Virtual Machine Manager.
- PVE1 : `192.168.122.27`.
- PVE2 : `192.168.122.37`.
- Cluster : `mycluser`.

## 2. Télécharger les images ISO

Télécharger les images ISO nécessaires :

- Alpine Linux : https://alpinelinux.org/downloads/
- Ubuntu Server : https://ubuntu.com/download/server

Dans l'interface Web Proxmox :

1. Sélectionner le nœud PVE1.
2. Ouvrir le stockage `local`.
3. Accéder à `ISO Images`.
4. Cliquer sur `Upload` ou télécharger directement l'image ISO.

Répéter l'opération sur PVE2 pour Ubuntu Server.

**Remarque :** si un stockage ISO partagé est disponible, les images peuvent être utilisées depuis les deux nœuds sans duplication.

## 3. Création de la VM Alpine Linux

Depuis l'interface Web Proxmox de PVE1 :

1. Sélectionner PVE1.
2. Cliquer sur `Create VM`.
3. Configurer les paramètres suivants.

| Paramètre | Configuration |
|---|---|
| Name | Alpine-VM |
| Node | PVE1 |
| OS | Alpine Linux ISO |
| CPU | 1 à 2 cores |
| RAM | 1024 à 2048 MB |
| Disk | 8 à 16 GB |
| Network | VirtIO (paravirtualized) |

Conserver les autres paramètres par défaut si aucune modification n'est nécessaire.

Cliquer sur `Finish`, puis démarrer la VM.

### Installation d'Alpine Linux

Ouvrir la console de la VM.

Se connecter avec l'utilisateur `root` (initialement sans mot de passe dans l'environnement live), puis lancer :

```bash
setup-alpine
```

Suivre l'assistant d'installation :

- Configurer le clavier.
- Définir le hostname : `alpine-vm`.
- Configurer le réseau.
- Définir le mot de passe root.
- Sélectionner le dépôt Alpine.
- Choisir le disque d'installation.
- Confirmer l'installation.

Redémarrer la VM après l'installation et retirer l'ISO du lecteur virtuel.

Vérifier l'installation :

```bash
cat /etc/alpine-release
hostname
ip addr
```

## 4. Création de la VM Ubuntu Server

Depuis l'interface Web Proxmox de PVE2 :

1. Sélectionner PVE2.
2. Cliquer sur `Create VM`.
3. Configurer les paramètres suivants.

| Paramètre | Configuration |
|---|---|
| Name | Ubuntu-VM |
| Node | PVE2 |
| OS | Ubuntu Server ISO |
| CPU | 2 cores |
| RAM | 2048 à 4096 MB |
| Disk | 20 à 25 GB |
| Network | VirtIO (paravirtualized) |

Cliquer sur `Finish`, puis démarrer la VM.

### Installation d'Ubuntu Server

Ouvrir la console et suivre l'assistant d'installation :

1. Choisir la langue.
2. Configurer le clavier.
3. Configurer le réseau.
4. Définir le hostname : `ubuntu-vm`.
5. Créer un utilisateur.
6. Configurer le disque.
7. Activer l'installation du serveur OpenSSH.

Une fois l'installation terminée, redémarrer la VM et retirer l'ISO.

Vérifier :

```bash
hostname
ip addr
lsb_release -a
```

## 5. Configuration du réseau

L'objectif est de permettre aux deux machines virtuelles de communiquer entre elles et avec les autres machines de l'infrastructure.

Sur Alpine :

```bash
ip addr
ip route
```

Sur Ubuntu :

```bash
ip addr
ip route
```

Vérifier que les deux VMs disposent d'adresses IP accessibles sur le réseau virtuel.

Tester la communication :

Depuis Alpine :

```bash
ping IP_UBUNTU
```

Depuis Ubuntu :

```bash
ping IP_ALPINE
```

**Résultat attendu :** les deux machines virtuelles communiquent correctement.

## 6. Configuration de SSH

SSH permet d'administrer les machines virtuelles à distance depuis l'hôte ou une autre machine du réseau.

### Sur Alpine Linux

Installer OpenSSH :

```bash
apk update
apk add openssh
```

Activer le service :

```bash
rc-update add sshd default
rc-service sshd start
```

Vérifier :

```bash
rc-service sshd status
```

### Sur Ubuntu Server

Installer OpenSSH :

```bash
sudo apt update
sudo apt install openssh-server -y
```

Activer le service :

```bash
sudo systemctl enable --now ssh
```

Vérifier :

```bash
sudo systemctl status ssh
```

### Tester les connexions SSH

Depuis Ubuntu vers Alpine :

```bash
ssh root@IP_ALPINE
```

Depuis Alpine vers Ubuntu :

```bash
ssh utilisateur@IP_UBUNTU
```

**Résultat attendu :** les connexions SSH fonctionnent correctement.

## 7. Installation de QEMU Guest Agent

QEMU Guest Agent permet à Proxmox de communiquer avec le système invité et facilite certaines opérations d'administration.

### Sur Alpine Linux

Installer l'agent :

```bash
apk add qemu-guest-agent
```

Activer le service :

```bash
rc-update add qemu-guest-agent default
rc-service qemu-guest-agent start
```

Vérifier :

```bash
rc-service qemu-guest-agent status
```

### Sur Ubuntu Server

Installer l'agent :

```bash
sudo apt update
sudo apt install qemu-guest-agent -y
```

Activer le service :

```bash
sudo systemctl enable --now qemu-guest-agent
```

Vérifier :

```bash
sudo systemctl status qemu-guest-agent
```

### Activer l'agent dans Proxmox

Pour chaque VM :

1. Sélectionner la VM.
2. Aller dans `Options`.
3. Sélectionner `QEMU Guest Agent`.
4. Cliquer sur `Edit`.
5. Activer l'option.
6. Redémarrer la VM si nécessaire.

Vérifier que l'agent fonctionne depuis Proxmox.

## 8. Mise à jour des systèmes

Sur Alpine :

```bash
apk update
apk upgrade
```

Sur Ubuntu :

```bash
sudo apt update
sudo apt upgrade -y
```

## 9. Tests et validation

Effectuer les tests suivants :

| Test | Résultat attendu |
|---|---|
| Installation Alpine | Réussie |
| Installation Ubuntu Server | Réussie |
| Attribution des IP | Fonctionnelle |
| Ping Alpine → Ubuntu | Réussi |
| Ping Ubuntu → Alpine | Réussi |
| SSH vers Alpine | Fonctionnel |
| SSH vers Ubuntu | Fonctionnel |
| QEMU Guest Agent | Actif |
| Mise à jour des systèmes | Réussie |

## 10. Résultat final

À la fin de cette étape, l'infrastructure contient deux machines virtuelles invitées :

- Alpine Linux, hébergée sur PVE1.
- Ubuntu Server, hébergée sur PVE2.

Les deux systèmes sont installés, configurés et accessibles à distance par SSH.

Ils pourront maintenant être utilisés pour les prochaines étapes du projet : configuration du stockage partagé, migration des machines virtuelles et étude de la haute disponibilité (HA).
