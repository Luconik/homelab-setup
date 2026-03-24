# EVE-NG Community sur Proxmox VE

Guide d'installation d'**EVE-NG Community Edition** sur un hôte **Proxmox VE**, incluant la configuration du mode promiscuous sur `vmbr0` et la connectivité avec les nodes AOS-CX.

> 🇫🇷 Documentation principale en français — un résumé en anglais est disponible dans chaque section.  
> 🇬🇧 Main documentation in French — an English summary is available in each section.

---

## Prérequis

| Élément | Configuration utilisée |
|---------|----------------------|
| Hôte | Proxmox VE |
| vCPU alloués | 12 vCPU |
| RAM allouée | 24 576 Mo (24 Go) |
| Disque | 200 Go |
| ISO | EVE-NG Community 6.2.0-4 |

> 💡 Pour dimensionner correctement la VM selon tes nodes, utilise le [nodes per lab calculator](https://drive.google.com/file/d/1Rbu7KDNSNuWiv_AphWx0vCek8CKVB1WI/view) disponible dans le [EVE-NG Community Cookbook](https://www.eve-ng.net/wp-content/uploads/2020/05/EVE-Comm-BOOK-1.08-2020.pdf).

Exemple : pour 6 switches AOS-CX Simulator 10.16, le calculateur donne **24 576 Mo RAM** et **12 vCPU**.

![Calculateur EVE-NG — résultat AOS-CX](images/EVE-CALC2_0_aoscx_Result.png)

> **EN** — Use the EVE-NG nodes per lab calculator to size your VM. For 6 AOS-CX Simulator nodes, the minimum is 24GB RAM and 12 vCPUs.

---

## Étape 1 — Création de la VM sur Proxmox

Depuis la console Proxmox, cliquer sur **"Create VM"**.

![Create VM](images/CreateVM.png)

### 1.1 — General

Renseigner le **Node**, le **VM ID** et le **Name** de la VM, puis cliquer **Next**.

![General](images/Create_General.png)

### 1.2 — OS

Sélectionner l'image **ISO EVE-NG Community**, puis cliquer **Next**.

![OS — Sélection ISO](images/Create_OS.png)

### 1.3 — System

Cocher **Qemu Agent**, puis cliquer **Next**.

![System — Qemu Agent](images/Create_System.png)

### 1.4 — Disk

Sélectionner le **storage cible** et définir la taille du disque (200 Go recommandés), puis cliquer **Next**.

![Disk — Storage et taille](images/Create_Disk.png)

### 1.5 — CPU

Définir le nombre de **Cores** et sélectionner le type **`host`** pour bénéficier de l'Intel VT-x (nested virtualization), puis cliquer **Next**.

![CPU — Cores et type host](images/Create_CPU.png)

> ⚠️ Le type **`host`** est indispensable — il expose les extensions de virtualisation Intel VT-x directement à la VM. Sans ça, les nodes EVE-NG ne démarreront pas.

### 1.6 — Memory

Définir la quantité de RAM (24 576 Mo pour 6 nodes AOS-CX), puis cliquer **Next**.

![Memory — RAM](images/Create_Memory.png)

### 1.7 — Network

Laisser les paramètres réseau par défaut, puis cliquer **Next**.

![Network — Défaut](images/Create_Network.png)

### 1.8 — Confirm

Vérifier les paramètres et cliquer **Finish**.

> **EN** — Create the VM in Proxmox: select the EVE-NG ISO, enable Qemu Agent, set CPU type to "host" for nested virtualization, allocate disk/RAM per the calculator, leave network defaults.

---

## Étape 2 — Installation d'EVE-NG

Démarrer la VM. Le GRUB EVE-NG s'affiche.

### 2.1 — Menu GRUB

Sélectionner **"Install EVE-NG Community 6.2.0-4"** et appuyer sur Entrée.

![GRUB — Install EVE-NG Community](images/EVE-NG_Installation-Step1.png)

### 2.2 — Choix de la langue

Sélectionner **Français**.

![Choix de la langue](images/EVE-NG_Installation-Step2.png)

### 2.3 — Configuration du clavier

- **Disposition** : French
- **Variante** : French (Macintosh) ← adapter selon ton clavier
- Cliquer **Terminé**

![Configuration clavier](images/EVE-NG_Installation-Step3.png)

### 2.4 — Confirmation de l'installation

Confirmer l'action en cliquant **Continuer**.

![Confirmer l'installation](images/EVE-NG_Installation-Step4.png)

L'installation se lance. Patienter jusqu'au redémarrage automatique.

> **EN** — Boot the VM, select EVE-NG Community in GRUB, choose language and keyboard, confirm. The system reboots automatically.

---

## ⚠️ Problèmes connus

### Erreur "Bad shim signature"

**Cause** : Le Secure Boot UEFI est activé sur la VM.

**Solution Proxmox** : Dans les paramètres de la VM → **Hardware** → **BIOS** → passer de `OVMF (UEFI)` à `SeaBIOS`.

**Solution alternative** : Depuis le GRUB → **"Advanced options for Ubuntu"** → sélectionner la version précédente du noyau.

> **EN** — If you see "Bad shim signature", switch the VM BIOS from OVMF (UEFI) to SeaBIOS in Proxmox hardware settings, or boot with the previous kernel version from GRUB advanced options.

---

## Étape 3 — Configuration initiale (Setup Wizard)

Après redémarrage, la VM affiche l'invite de login EVE-NG.

![Première connexion](images/EVE-NG-First_connection.png)

Se connecter avec :
- **Login** : `root`
- **Mot de passe** : `eve`

Le wizard de configuration se lance automatiquement.

### 3.1 — Mot de passe root

![Setup — Mot de passe root](images/EVE-NG-Setup-step1.png)
![Setup — Confirmer le mot de passe](images/EVE-NG-Setup-step2.png)

### 3.2 — Hostname

![Setup — Hostname](images/EVE-NG-Setup-step3.png)

### 3.3 — Nom de domaine DNS

![Setup — DNS domain name](images/EVE-NG-Setup-step4.png)

### 3.4 — Mode IP

Sélectionner **static** (recommandé).

![Setup — DHCP ou Static](images/EVE-NG-Setup-step5.png)

### 3.5 — Adresse IP, masque, passerelle, DNS, NTP, Proxy

![Setup — Adresse IP](images/EVE-NG-Setup-step6.png)
![Setup — Masque](images/EVE-NG-Setup-step7.png)
![Setup — Passerelle](images/EVE-NG-Setup-step8.png)
![Setup — DNS primaire](images/EVE-NG-Setup-step9.png)
![Setup — DNS secondaire](images/EVE-NG-Setup-step10.png)
![Setup — NTP](images/EVE-NG-Setup-step11.png)
![Setup — Proxy](images/EVE-NG-Setup-step12.png)

> 💡 Si le wizard a planté ou qu'un paramètre est erroné, le relancer avec :
> ```bash
> rm -f /opt/ovf/.configured && su -
> ```

> **EN** — After reboot, log in as root/eve. Configure hostname, domain, static IP, gateway, DNS, NTP, and proxy (direct connection).

---

## Étape 4 — Mises à jour

![Connexion root et mise à jour](images/EVE-NG-Update.png)

```bash
apt update && apt upgrade -y
```

---

## Étape 5 — Configuration du mode Promiscuous sur vmbr0

**Spécificité Proxmox** : pour permettre aux nodes EVE-NG d'accéder au réseau physique et joindre les switches AOS-CX depuis ton poste, il faut activer le mode promiscuous sur `vmbr0`.

Éditer le fichier de configuration réseau de l'hôte Proxmox :

```bash
nano /etc/network/interfaces
```

Ajouter la ligne `post-up` sur le bridge `vmbr0` :

```
auto lo
iface lo inet loopback

iface eno1 inet manual

auto vmbr0
iface vmbr0 inet static
        address 10.224.100.111/24
        gateway 10.224.100.1
        bridge-ports eno1
        bridge-stp off
        bridge-fd 0
        post-up ip link set dev vmbr0 promisc on    # <--- AJOUTER CETTE LIGNE

iface wlp0s20f3 inet manual

source /etc/network/interfaces.d/*
```

Appliquer sans redémarrer :

```bash
ifreload -a
```

> **EN** — On Proxmox, enable promiscuous mode on vmbr0 by adding "post-up ip link set dev vmbr0 promisc on" to /etc/network/interfaces. This allows EVE-NG nodes to communicate with devices on the physical network.

---

## Étape 6 — Configuration réseau AOS-CX

Une fois le node AOS-CX démarré dans EVE-NG, configurer une IP de management pour y accéder depuis ton poste :

```bash
configure terminal
ip mgmt
ip static 10.224.100.242/24
default-gateway 10.224.100.1
```

> **EN** — Configure a management IP on the AOS-CX node to allow SSH/web access from your workstation through the Proxmox vmbr0 bridge.

---

## Étape 7 — Installation des images réseau

### 7.1 — HPE Aruba AOS-CX

Uploader le fichier `.ova` AOS-CX sur la VM EVE-NG (via SCP ou SFTP), puis :

```bash
# Créer un dossier de travail et extraire l'OVA
mkdir ~/aos-cx && cd ~/aos-cx
# Copier l'OVA dans ce dossier au préalable
tar xvf AOS-CX_10_15_0005.ova

# Convertir le VMDK en qcow2
/opt/qemu/bin/qemu-img convert -f vmdk -O qcow2 \
  arubaoscx-disk-image-genericx86-p4-*.vmdk virtioa.qcow2

# Créer le dossier destination et déplacer l'image
mkdir /opt/unetlab/addons/qemu/aruba_aoscx-10.15
mv virtioa.qcow2 /opt/unetlab/addons/qemu/aruba_aoscx-10.15/

# Nettoyer et corriger les permissions
cd && rm -rf ~/aos-cx
/opt/unetlab/wrappers/unl_wrapper -a fixpermissions
```

> 💡 Adapter le nom du dossier selon la version (ex : `aruba_aoscx-10.16`).

### 7.2 — Juniper vJunos Switch

```bash
mkdir /opt/unetlab/addons/qemu/vjunosswitch-23.2R1.14
cd /opt/unetlab/addons/qemu/vjunosswitch-23.2R1.14

mv vJunos-switch-23.2R1.14.qcow2 virtioa.qcow2

/opt/unetlab/wrappers/unl_wrapper -a fixpermissions
```

### 7.3 — Juniper vJunos Evolved (EVO)

```bash
mkdir /opt/unetlab/addons/qemu/vjunosevo-23.2R2.21
cd /opt/unetlab/addons/qemu/vjunosevo-23.2R2.21

mv vJunosEvolved-23.2R2.21-EVO.qcow2 virtioa.qcow2

/opt/unetlab/wrappers/unl_wrapper -a fixpermissions
```

### 7.4 — Juniper vJunos Router

```bash
mkdir /opt/unetlab/addons/qemu/vJunos-router-23.2R1.15
cd /opt/unetlab/addons/qemu/vJunos-router-23.2R1.15

mv vJunos-router-23.2R1.15.qcow2 virtioa.qcow2

/opt/unetlab/wrappers/unl_wrapper -a fixpermissions
```

> ⚠️ **Règle impérative** : le fichier image doit toujours s'appeler `virtioa.qcow2` dans son dossier, quelle que soit la plateforme.

> **EN** — All network appliance images must be placed in `/opt/unetlab/addons/qemu/[image-folder]/` and renamed to `virtioa.qcow2`. Always run fixpermissions after adding any image. AOS-CX OVA files must first be extracted and converted from VMDK to qcow2 format.

---

## Ressources utiles

- 📖 [Documentation officielle EVE-NG](https://www.eve-ng.net/index.php/documentation/)
- 📖 [EVE-NG Community Cookbook](https://www.eve-ng.net/wp-content/uploads/2020/05/EVE-Comm-BOOK-1.08-2020.pdf)
- 🧮 [Nodes per lab calculator](https://drive.google.com/file/d/1Rbu7KDNSNuWiv_AphWx0vCek8CKVB1WI/view)
- 🔗 [Guide EVE-NG sur ESXi](../esxi/README.md)
- 🔗 [Repo netdevops](https://github.com/Luconik/netdevops)

---

> Les guides sont librement réutilisables avec mention de l'auteur (CC BY 4.0).
