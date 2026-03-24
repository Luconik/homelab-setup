# Ubuntu Server 24.04 LTS sur Proxmox VE

Guide d'installation et de configuration d'**Ubuntu Server 24.04 LTS** comme VM sur **Proxmox VE 9.1**.

> 🇫🇷 Documentation principale en français — un résumé en anglais est disponible dans chaque section.  
> 🇬🇧 Main documentation in French — an English summary is available in each section.

---

## Prérequis

| Élément | Configuration utilisée |
|---------|----------------------|
| Hôte | Proxmox VE 9.1.4 |
| vCPU | 2 cores |
| RAM | 4 096 Mo (4 Go) |
| Disque | 60 Go |
| ISO | Ubuntu Server 24.04 LTS (noble) |
| Réseau | Bridge vmbr0 — VirtIO paravirtualized |

> **EN** — Proxmox VE 9.1, 2 vCPUs, 4GB RAM, 60GB disk, Ubuntu Server 24.04 LTS ISO, vmbr0 VirtIO network bridge.

---

## Étape 1 — Téléchargement de l'ISO

1. Aller sur [ubuntu.com/download/server](https://ubuntu.com/download/server)
2. Télécharger **Ubuntu Server 24.04 LTS**
3. Uploader l'ISO dans Proxmox : **Datacenter → Storage → local → ISO Images → Upload**

> **EN** — Download Ubuntu Server 24.04 LTS and upload the ISO to your Proxmox local storage via the web interface.

---

## Étape 2 — Création de la VM dans Proxmox

Depuis la console Proxmox, cliquer sur **"Create VM"** en haut à droite.

![Console Proxmox — Create VM](images/CreateVM.png)

### 2.1 — General

Renseigner le **Node** (`pve1`), le **VM ID** et le **Name** de la VM (ex : `luconik-automation`), puis cliquer **Next**.

![General](images/Create_General.png)

### 2.2 — OS

Sélectionner l'ISO Ubuntu Server 24.04 LTS, **Guest OS Type : Linux**, **Version : 6.x - 2.6 Kernel**, puis cliquer **Next**.

![OS — Sélection ISO](images/Create_OS.png)

### 2.3 — System

Laisser **BIOS : Default (SeaBIOS)**, cocher ✅ **Qemu Agent**, puis cliquer **Next**.

![System — Qemu Agent](images/Create_System.png)

### 2.4 — Disks

Sélectionner le storage (`local-lvm`), définir la taille (60 Go), puis cliquer **Next**.

![Disk](images/Create_Disk.png)

### 2.5 — CPU

Définir **2 Cores**, **Type : x86-64-v2-AES**, puis cliquer **Next**.

![CPU](images/Create_CPU.png)

### 2.6 — Memory

Définir **4096 MiB** (4 Go), puis cliquer **Next**.

![Memory](images/Create_Memory.png)

### 2.7 — Network

Laisser le bridge **vmbr0**, modèle **VirtIO (paravirtualized)**, puis cliquer **Next**.

![Network](images/Create_Network.png)

### 2.8 — Confirm

Vérifier les paramètres et cliquer **Finish**.

![Confirm](images/Create_Confirm.png)

> **EN** — Create VM with Ubuntu Linux guest, SeaBIOS, Qemu Agent enabled, 60GB SCSI disk on local-lvm, 2 cores, 4GB RAM, vmbr0 VirtIO network.

---

## Étape 3 — Installation Ubuntu Server

Démarrer la VM et ouvrir la console Proxmox (bouton **Console** dans le menu de la VM).

### 3.1 — Mise à jour de l'installeur

Si une mise à jour de l'installeur est disponible, sélectionner **"Update to the new installer"** ou **"Continue without updating"** selon ta préférence.

![Mise à jour installeur](images/Install_Update_Installer.png)

### 3.2 — Langue

Sélectionner **English** (langue par défaut sur Ubuntu 24.04), puis appuyer sur Entrée.

![Langue](images/Install_Language.png)

### 3.3 — Configuration du clavier

Sélectionner la disposition clavier adaptée à ton matériel (ex : **English (US) — English (Macintosh)**), puis **Done**.

![Clavier](images/Install_Keyboard.png)

### 3.4 — Type d'installation

Sélectionner **"Ubuntu Server"** (installation standard), puis **Done**.

![Type installation](images/Install_Type.png)

### 3.5 — Configuration réseau

L'interface **ens18** est détectée automatiquement en DHCP. Laisser tel quel pour l'installation — on configurera l'IP statique après.

![Réseau — ens18 DHCP](images/Install_Network.png)

### 3.6 — Proxy

Laisser vide si pas de proxy, puis **Done**.

![Proxy](images/Install_Proxy.png)

### 3.7 — Miroir APT

Le miroir français `http://fr.archive.ubuntu.com/ubuntu/` est détecté automatiquement. Laisser par défaut et cliquer **Done**.

![Miroir APT](images/Install_Mirror.png)

### 3.8 — Configuration du disque

Sélectionner **"Use an entire disk"** avec LVM, puis **Done**.

![Disque](images/Install_Disk.png)

Confirmer le résumé du partitionnement → **Done → Continue**.

![Résumé disque](images/Install_Disk_Summary.png)

### 3.9 — Profil utilisateur

Renseigner les informations du compte :

| Champ | Exemple |
|-------|---------|
| Your name | `luconik` |
| Your server's name | `luconik-automation` |
| Pick a username | `luconik` |
| Password | Choisir un mot de passe fort |

![Profil](images/Install_Profile.png)

### 3.10 — Ubuntu Pro

Sélectionner **"Skip for now"**, puis **Continue**.

![Ubuntu Pro](images/Install_Ubuntu_Pro.png)

### 3.11 — SSH

Cocher ✅ **"Install OpenSSH server"**, puis **Done**.

![SSH](images/Install_SSH.png)

### 3.12 — Snaps additionnels

Ne rien sélectionner, puis **Done**.

![Snaps](images/Install_Snaps.png)

### 3.13 — Installation en cours

Patienter jusqu'à la fin de l'installation.

![Installation en cours](images/Install_Progress.png)

### 3.14 — Redémarrage

Quand **"Installation complete!"** s'affiche, cliquer **"Reboot Now"**.

![Installation complète — Reboot Now](images/Install_Reboot.png)

> **EN** — Ubuntu Server 24.04 installs in English by default. Update the installer if prompted, configure keyboard, select Ubuntu Server install type, leave network on DHCP, use entire disk with LVM, create user, skip Ubuntu Pro, enable OpenSSH, skip snaps. Reboot when done.

---

## Étape 4 — Configuration IP statique

Après redémarrage, se connecter avec les identifiants créés lors de l'installation.

### 4.1 — Identifier l'interface réseau

```bash
ip a
# Interface détectée : ens18
```

### 4.2 — Éditer la configuration Netplan

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Remplacer le contenu par une configuration IP statique :

```yaml
network:
  version: 2
  ethernets:
    ens18:
      addresses:
        - 10.224.100.X/24          # ton IP statique
      routes:
        - to: default
          via: 10.224.100.1        # ta gateway
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
      dhcp4: false
```

Appliquer et vérifier :

```bash
sudo netplan apply
ip a
ping 8.8.8.8 -c 4
```

> **EN** — Edit /etc/netplan/00-installer-config.yaml to set a static IP on ens18. Apply with "sudo netplan apply" and verify connectivity.

---

## Étape 5 — Configuration SSH

### 5.1 — Connexion depuis ton Mac

```bash
ssh luconik@[IP_SERVEUR]
```

### 5.2 — Authentification par clé SSH (recommandé)

Sur ton Mac, générer une paire de clés si besoin :

```bash
ssh-keygen -t ed25519 -C "nicolas@culetto.fr"
```

Copier la clé publique sur le serveur :

```bash
ssh-copy-id luconik@[IP_SERVEUR]
```

### 5.3 — Désactiver l'authentification par mot de passe (optionnel)

```bash
sudo nano /etc/ssh/sshd_config
# Modifier :
# PasswordAuthentication no
# PubkeyAuthentication yes

sudo systemctl restart ssh
```

> **EN** — Connect via SSH from your Mac. Use SSH key authentication for better security. Optionally disable password login in /etc/ssh/sshd_config.

---

## Étape 6 — Post-installation

### 6.1 — Mises à jour

```bash
sudo apt update && sudo apt upgrade -y
```

### 6.2 — Outils de base

```bash
sudo apt install -y \
  curl \
  wget \
  git \
  vim \
  htop \
  net-tools \
  unzip \
  ca-certificates \
  gnupg \
  lsb-release
```

### 6.3 — Qemu Guest Agent

Pour que Proxmox puisse afficher l'IP de la VM et effectuer des snapshots propres :

```bash
sudo apt install -y qemu-guest-agent
sudo systemctl enable --now qemu-guest-agent
```

### 6.4 — Vérification finale

```bash
ip a                                    # IP statique configurée
ping 8.8.8.8 -c 4                       # Connectivité internet
sudo systemctl status ssh               # SSH actif
sudo systemctl status qemu-guest-agent  # Qemu agent actif
```

> **EN** — Update packages, install basic tools, and enable qemu-guest-agent for Proxmox integration (VM IP visible in UI, clean snapshots).

---

## Ressources utiles

- 📖 [Documentation officielle Ubuntu Server](https://ubuntu.com/server/docs)
- 📖 [Documentation Netplan](https://netplan.io/reference)
- 🔗 [Guide EVE-NG sur Proxmox](../eve-ng/proxmox/README.md)
- 🔗 [Guide Docker n8n](../docker/n8n/README.md)

---

> Les guides sont librement réutilisables avec mention de l'auteur (CC BY 4.0).
