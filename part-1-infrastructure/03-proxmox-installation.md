# 03 – Installation de Proxmox VE et d'Ubuntu Server

## 1. Objectif

Après la préparation de l'environnement VirtualBox, cette étape consiste à installer les systèmes d'exploitation nécessaires à notre infrastructure de virtualisation.

Nous allons installer :

- Proxmox VE sur PVE1.
- Proxmox VE sur PVE2.
- Ubuntu Server sur STORAGE-VM.

Les deux serveurs Proxmox constitueront les futurs nœuds du cluster. Le serveur STORAGE-VM hébergera les services NFS et Corosync QNetd.

Cette étape permet de :
- Installer Proxmox VE sur les deux machines virtuelles.
- Configurer les paramètres système des nœuds.
- Définir les noms d'hôtes et les adresses IP de gestion.
- Installer Ubuntu Server sur STORAGE-VM.
- Vérifier l'accès aux interfaces Web Proxmox.
- Vérifier le fonctionnement initial des trois machines.

## 2. Environnement de travail

### 2.1. Configuration des machines

| Paramètre | PVE1 | PVE2 | STORAGE-VM |
|---|---|---|---|
| Système | Proxmox VE | Proxmox VE | Ubuntu Server |
| RAM | 4 Go | 4 Go | 1 Go |
| CPU | 2 vCPU | 2 vCPU | 1 vCPU |
| Disque | 50 Go | 50 Go | 20 Go |
| Réseau 1 | Host-only | Host-only | Host-only |
| Réseau 2 | Internal Network | Internal Network | Internal Network |

### 2.2. Plan d'adressage

Les adresses suivantes seront utilisées dans notre laboratoire.

| Machine | Nom d'hôte | IP de gestion |
|---|---|---|
| PVE1 | pve1.proxmox.lab | 192.168.56.11 |
| PVE2 | pve2.proxmox.lab | 192.168.56.12 |
| STORAGE-VM | storage.proxmox.lab | 192.168.56.20 |

Le réseau interne utilisera les adresses suivantes :

| Machine | IP réseau interne |
|---|---|
| PVE1 | 10.10.10.11 |
| PVE2 | 10.10.10.12 |
| STORAGE-VM | 10.10.10.20 |

**Important :** les adresses IP du réseau interne seront configurées dans `04-network-configuration.md`.

Les noms d'hôtes et les adresses IP doivent rester identiques dans tous les guides.

## 3. Préparation des images ISO

Avant de commencer, vérifier que les images ISO sont disponibles sur Windows 11.

| Image ISO | Utilisation |
|---|---|
| Proxmox VE ISO | Installation de PVE1 et PVE2 |
| Ubuntu Server ISO | Installation de STORAGE-VM |

Téléchargements officiels :

- Proxmox VE : https://www.proxmox.com/en/downloads
- Ubuntu Server : https://ubuntu.com/download/server

Pour ce laboratoire, utiliser la même version de Proxmox VE sur les deux nœuds.

**Remarque :** l'installation de Proxmox VE à partir de son ISO officiel installe également le système Debian nécessaire et les composants Proxmox. Il n'est pas nécessaire d'installer Debian séparément.

## 4. Installation de Proxmox VE sur PVE1

### 4.1. Démarrage sur l'image ISO

**Emplacement : VirtualBox – Windows 11**

1. Arrêter PVE1 si elle est déjà démarrée.
2. Sélectionner PVE1 dans VirtualBox.
3. Accéder à Configuration → Stockage.
4. Sélectionner le lecteur optique virtuel.
5. Choisir le fichier ISO Proxmox VE.
6. Vérifier que l'ordre de démarrage permet de démarrer sur le lecteur optique.
7. Démarrer PVE1.

L'écran d'installation Proxmox VE apparaît.

Sélectionner :

`Install Proxmox VE (Graphical)`

Appuyer sur Entrée.

### 4.2. Acceptation de la licence

L'installateur affiche le contrat de licence.

1. Lire les informations présentées.
2. Cliquer sur I agree.
3. Continuer vers la configuration du disque.

### 4.3. Sélection du disque

Sélectionner le disque virtuel de 50 Go créé précédemment.

Configuration proposée :

| Paramètre | Valeur |
|---|---|
| Disque cible | Disque virtuel de 50 Go |
| Système de fichiers | ext4 |
| Installation | Disque complet |

Pour ce laboratoire, le système de fichiers ext4 avec LVM constitue un choix simple pour le stockage local.

**Attention :** l'installation efface les données du disque sélectionné. Vérifier qu'il s'agit bien du disque virtuel de PVE1 et non d'un disque de Windows.

### 4.4. Configuration régionale

Configurer les paramètres suivants :

| Paramètre | Valeur |
|---|---|
| Pays | Morocco |
| Fuseau horaire | Africa/Casablanca |
| Clavier | French |

Ces paramètres peuvent être adaptés selon la configuration souhaitée.

### 4.5. Configuration du mot de passe

Définir un mot de passe pour le compte administrateur `root`.

Exemple de configuration :

- Utilisateur : root
- Mot de passe : choisir un mot de passe sécurisé.
- E-mail : adresse e-mail de l'administrateur.

Conserver le mot de passe : il sera nécessaire pour accéder à l'interface Web et administrer le nœud.

### 4.6. Configuration réseau

Cette étape est particulièrement importante.

L'installateur doit utiliser l'interface connectée au réseau Host-only pour permettre à Windows d'accéder à l'interface Web Proxmox.

Paramètres à configurer :

| Paramètre | Valeur |
|---|---|
| Hostname | pve1.proxmox.lab |
| IP Address | 192.168.56.11 |
| Netmask | 255.255.255.0 |
| Gateway | Aucune |
| DNS Server | 192.168.56.1 |

Le réseau Host-only utilisé dans notre laboratoire ne dispose pas nécessairement d'une passerelle Internet. Ne pas inventer une adresse de passerelle : laisser le champ vide si l'installateur le permet.

Pour le DNS, utiliser un serveur DNS accessible si nécessaire. L'adresse 192.168.56.1 ne doit être utilisée que si un service DNS y est réellement disponible.

**Important :** identifier correctement l'interface réseau avant de valider l'installation. Dans VirtualBox, l'adresse MAC de chaque adaptateur peut être consultée dans Configuration → Réseau → Avancé.

### 4.7. Lancement de l'installation

1. Vérifier le résumé de configuration.
2. Cliquer sur Install.
3. Attendre la fin de l'installation.
4. Redémarrer la machine virtuelle.
5. Retirer l'image ISO du lecteur virtuel si nécessaire.

PVE1 démarre maintenant sur son système Proxmox installé.

**Capture d'écran :**

`part-1-infrastructure/screenshots/09-pve1-installation.png`

Légende : Installation de Proxmox VE sur le premier nœud.

## 5. Installation de Proxmox VE sur PVE2

La procédure est identique à celle de PVE1.

### 5.1. Démarrage sur l'image ISO

1. Arrêter PVE2.
2. Sélectionner PVE2 dans VirtualBox.
3. Monter la même image ISO Proxmox VE.
4. Démarrer la machine.
5. Sélectionner Install Proxmox VE (Graphical).

### 5.2. Configuration du disque

Sélectionner le disque virtuel de 50 Go appartenant à PVE2.

Utiliser le même système de fichiers que sur PVE1 : ext4 avec LVM.

### 5.3. Configuration régionale

Utiliser les mêmes paramètres :

- Pays : Morocco.
- Fuseau horaire : Africa/Casablanca.
- Clavier : French.

### 5.4. Configuration du mot de passe

Définir le mot de passe administrateur `root`.

Il est recommandé d'utiliser un mot de passe sécurisé pour PVE2.

### 5.5. Configuration réseau

Utiliser les paramètres suivants :

| Paramètre | Valeur |
|---|---|
| Hostname | pve2.proxmox.lab |
| IP Address | 192.168.56.12 |
| Netmask | 255.255.255.0 |
| Gateway | Aucune |
| DNS Server | Serveur DNS accessible |

Ne pas réutiliser l'adresse IP de PVE1.

### 5.6. Finalisation

1. Vérifier les paramètres.
2. Cliquer sur Install.
3. Attendre la fin de l'installation.
4. Redémarrer PVE2.
5. Retirer l'image ISO.

**Capture d'écran :**

`part-1-infrastructure/screenshots/10-pve2-installation.png`

Légende : Installation de Proxmox VE sur le deuxième nœud.

## 6. Premier démarrage et accès aux interfaces Web Proxmox

Après l'installation, chaque serveur Proxmox affiche une adresse permettant d'accéder à son interface Web.

### 6.1. Accès à PVE1

**Emplacement : Windows 11 – Navigateur Web**

Ouvrir :

`https://192.168.56.11:8006`

Une alerte de certificat SSL peut apparaître. Il s'agit du certificat autosigné utilisé par défaut.

Dans cet environnement de laboratoire, accepter l'exception de sécurité après avoir vérifié que l'adresse correspond bien à PVE1.

Se connecter avec :

| Paramètre | Valeur |
|---|---|
| Utilisateur | root |
| Mot de passe | Mot de passe défini pendant l'installation |
| Realm | Linux PAM standard |
| Langue | Français ou English |

### 6.2. Accès à PVE2

Ouvrir :

`https://192.168.56.12:8006`

Utiliser les identifiants administrateur de PVE2.

### 6.3. Vérification

Résultat attendu :

- L'interface Web de PVE1 est accessible.
- L'interface Web de PVE2 est accessible.
- Les deux nœuds affichent leur propre Datacenter.
- Aucun cluster n'est encore créé.

**Captures d'écran :**

- `part-1-infrastructure/screenshots/11-pve1-web-interface.png`
- `part-1-infrastructure/screenshots/12-pve2-web-interface.png`

## 7. Installation d'Ubuntu Server sur STORAGE-VM

STORAGE-VM hébergera les services de stockage partagé et de quorum.

### 7.1. Démarrage sur l'image ISO

**Emplacement : VirtualBox**

1. Arrêter STORAGE-VM.
2. Sélectionner la machine dans VirtualBox.
3. Accéder à Configuration → Stockage.
4. Monter l'image ISO Ubuntu Server.
5. Démarrer la machine virtuelle.

### 7.2. Installation d'Ubuntu Server

Dans le menu de démarrage, sélectionner l'installation Ubuntu Server.

Suivre les étapes de l'installateur :

1. Sélectionner la langue.
2. Configurer le clavier.
3. Choisir Ubuntu Server.
4. Configurer le réseau.
5. Sélectionner le disque virtuel.
6. Créer un utilisateur administrateur.
7. Définir un mot de passe.
8. Activer l'installation du serveur OpenSSH si cette option est proposée.

### 7.3. Configuration réseau

Pour le premier adaptateur Host-only, utiliser :

| Paramètre | Valeur |
|---|---|
| Nom d'hôte | storage |
| Adresse IP | 192.168.56.20 |
| Masque | 255.255.255.0 |
| Passerelle | Aucune |
| DNS | Serveur DNS accessible |

La configuration du deuxième adaptateur Internal Network sera effectuée dans le guide réseau.

### 7.4. Configuration du disque

Utiliser le disque virtuel de 20 Go.

Pour ce laboratoire, le partitionnement automatique proposé par Ubuntu Server est suffisant.

### 7.5. Installation et redémarrage

1. Vérifier le résumé de configuration.
2. Confirmer l'installation.
3. Attendre la copie des fichiers.
4. Redémarrer STORAGE-VM.
5. Retirer l'image ISO du lecteur virtuel.

**Capture d'écran :**

`part-1-infrastructure/screenshots/13-storage-ubuntu-installation.png`

## 8. Vérification des installations

Après l'installation des trois systèmes, effectuer les vérifications suivantes.

### 8.1. Vérification de PVE1

**Emplacement : Console PVE1**

Afficher le nom d'hôte :

```bash
hostnamectl
```

Afficher les interfaces réseau :

```bash
ip -br addr
```

Afficher les informations de stockage :

```bash
lsblk
```

### 8.2. Vérification de PVE2

**Emplacement : Console PVE2**

```bash
hostnamectl
ip -br addr
lsblk
```

### 8.3. Vérification de STORAGE-VM

**Emplacement : Terminal Ubuntu Server**

```bash
hostnamectl
```

Afficher les interfaces réseau :

```bash
ip -br addr
```

Vérifier le stockage :

```bash
lsblk
df -h
```

### 8.4. Vérification des interfaces Web

Depuis Windows 11, vérifier que les deux interfaces Proxmox sont accessibles :

```text
https://192.168.56.11:8006
https://192.168.56.12:8006
```

## 9. Mise à jour des systèmes

Une fois la connectivité Internet configurée, effectuer les mises à jour.

### Sur PVE1 et PVE2

```bash
apt update
apt dist-upgrade -y
```

### Sur STORAGE-VM

```bash
sudo apt update
sudo apt upgrade -y
```

**Remarque :** si le réseau Host-only ne fournit pas d'accès Internet, prévoir temporairement un adaptateur NAT dans VirtualBox pour les mises à jour. Il pourra être désactivé après utilisation. Ne pas modifier les deux réseaux permanents du laboratoire.

## 10. Vérification de la virtualisation imbriquée

Proxmox doit pouvoir accéder aux extensions de virtualisation du processeur pour exécuter ses propres machines virtuelles.

### Sur PVE1

```bash
egrep -c '(vmx|svm)' /proc/cpuinfo
```

### Sur PVE2

```bash
egrep -c '(vmx|svm)' /proc/cpuinfo
```

Un résultat supérieur à zéro indique que les extensions matérielles sont visibles.

Vérifier également :

```bash
ls -l /dev/kvm
```

Résultat attendu :

- Les extensions de virtualisation sont visibles.
- Le périphérique `/dev/kvm` est disponible.

Si `/dev/kvm` n'existe pas, vérifier la configuration de virtualisation imbriquée dans VirtualBox avant de poursuivre.

## 11. Dépannage

### Problème 1 : Impossible d'accéder à l'interface Web

Causes possibles :
- Mauvaise adresse IP.
- Mauvais adaptateur réseau.
- Interface Host-only incorrectement configurée.
- Service Web Proxmox indisponible.

Vérifier depuis la console Proxmox :

```bash
ip -br addr
systemctl status pveproxy
```

Tester ensuite la connectivité depuis Windows.

### Problème 2 : Adresse IP déjà utilisée

Deux machines ne doivent jamais utiliser la même adresse IP sur le même réseau.

Vérifier les adresses configurées :

```bash
ip -br addr
```

Chaque machine doit utiliser son adresse propre.

### Problème 3 : Proxmox ne détecte pas KVM

Vérifier :

```bash
egrep -c '(vmx|svm)' /proc/cpuinfo
ls -l /dev/kvm
```

Si la virtualisation matérielle n'est pas disponible, vérifier VT-x/AMD-V dans le BIOS/UEFI et la virtualisation imbriquée dans VirtualBox.

### Problème 4 : L'installation ne démarre pas

Causes possibles :
- Image ISO incorrecte.
- Image ISO non montée.
- Ordre de démarrage incorrect.
- Ressources insuffisantes.

Vérifier les paramètres de stockage et de démarrage de VirtualBox.

## 12. Vérification finale

Avant de passer à la configuration réseau, vérifier les éléments suivants :

- [ ] Proxmox VE est installé sur PVE1.
- [ ] Proxmox VE est installé sur PVE2.
- [ ] Ubuntu Server est installé sur STORAGE-VM.
- [ ] Les noms d'hôtes sont configurés.
- [ ] Les adresses IP de gestion sont configurées.
- [ ] Les interfaces Web Proxmox sont accessibles.
- [ ] Les mises à jour sont effectuées lorsque la connexion Internet est disponible.
- [ ] La virtualisation imbriquée est fonctionnelle.
- [ ] Le périphérique KVM est disponible sur les deux nœuds.

## 13. Résultat attendu

À la fin de cette étape, notre infrastructure contient trois systèmes opérationnels :

| Machine | Système installé | Fonction |
|---|---|---|
| PVE1 | Proxmox VE | Premier hyperviseur |
| PVE2 | Proxmox VE | Deuxième hyperviseur |
| STORAGE-VM | Ubuntu Server | Futur serveur NFS et QNetd |

Les deux interfaces Web Proxmox sont accessibles depuis Windows 11.

Les deux nœuds sont installés individuellement, mais ne forment pas encore un cluster.

## 14. Conclusion

Cette étape a permis d'installer les deux hyperviseurs Proxmox VE et le serveur Ubuntu supplémentaire.

Les systèmes sont maintenant prêts pour la configuration réseau.

La prochaine étape consiste à configurer les interfaces réseau, les adresses IP et la communication entre PVE1, PVE2 et STORAGE-VM.
