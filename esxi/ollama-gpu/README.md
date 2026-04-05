# GPU Passthrough RTX 3060 sous ESXi 8 + Ollama

> **Statut :** ✅ Fonctionnel  
> **Date :** Avril 2026  
> **Matériel :** ASUS DUAL RTX 3060 12GB OC V2 | ESXi 8.0 U3 | i9-10850K | 128GB RAM

---

## Contexte

Mise en place d'un serveur Ollama dédié (VM `ollama-gpu`) avec accélération GPU sur ESXi 8, en passant la RTX 3060 en PCI passthrough vers une VM Ubuntu. Ce guide documente les erreurs rencontrées et la solution finale.

---

## Architecture finale

```
ESXi 8.0 U3 (Z390 / i9-10850K / 128GB RAM)
  └── VM ollama-gpu (Ubuntu 24.04 LTS)
        ├── 8 vCPU
        ├── 32GB RAM (réservation totale obligatoire)
        ├── PCI passthrough : RTX 3060 12GB (0000:01:00.0)
        ├── Ollama → port 11434
        └── Modèles stockés sur Synology via NFS (/mnt/ollama-models/)
```

---

## Étape 1 — Activer le passthrough sur ESXi

### Via l'interface web (si disponible)

```
Host → Manage → Hardware → PCI Devices
→ RTX 3060 → Toggle Passthrough → ON
→ Reboot ESXi
```

> ⚠️ Sur certaines versions d'ESXi 8, le bouton "Basculer le relais" est grisé dans l'UI.  
> Dans ce cas, passer par la CLI SSH (voir ci-dessous).

### Via SSH ESXi (méthode fiable)

```bash
# Identifier l'adresse PCI de la carte
esxcli hardware pci list | grep -A5 -i nvidia

# Activer le passthrough (adapter l'adresse PCI)
esxcli hardware pci pcipassthru set -d 0000:01:00.0 -e true
esxcli hardware pci pcipassthru set -d 0000:01:00.1 -e true  # contrôleur audio HDMI

# Vérifier
esxcli hardware pci pcipassthru list | grep "01:00"
# Résultat attendu :
# 0000:01:00.0     true
# 0000:01:00.1     true
```

> Pas de reboot ESXi nécessaire si le passthrough était déjà configuré.

---

## Étape 2 — Créer la VM Ubuntu

### Paramètres VM dans ESXi

| Paramètre | Valeur |
|---|---|
| Nom | ollama-gpu |
| OS | Ubuntu Linux 64-bit |
| vCPU | 8 |
| RAM | 32 GB (**cocher "Reserve all guest memory"** — obligatoire pour PCI passthrough) |
| Disque | 100 GB |
| Réseau | vSwitch LAN habituel |

### Ajouter le GPU à la VM

```
VM Settings → Add other device → PCI Device
→ Sélectionner 0000:01:00.0 (RTX 3060)
→ Sélectionner 0000:01:00.1 (Audio HDMI)
```

### Paramètres VMX obligatoires

Dans **VM Options → Advanced → Edit Configuration**, ajouter :

```
hypervisor.cpuid.v0    = FALSE
pciPassthru.use64bitMMIO   = TRUE
pciPassthru.64bitMMIOSizeGB = 64
```

Vérifiable directement dans le fichier `.vmx` :

```bash
# Sur ESXi SSH
tail -5 /vmfs/volumes/<datastore>/ollama-gpu/ollama-gpu.vmx
# Résultat attendu :
# hypervisor.cpuid.v0 = "FALSE"
# pciPassthru.use64bitMMIO = "TRUE"
# pciPassthru.64bitMMIOSizeGB = "64"
```

---

## Étape 3 — Installer Ubuntu 24.04 + driver NVIDIA

### Installation de base

```bash
sudo apt update && sudo apt upgrade -y
```

### ⚠️ Problèmes rencontrés (pour mémoire)

| Tentative | Driver | Erreur | Cause |
|---|---|---|---|
| 1 | `nvidia-driver-570-open` | `NVRM: GSP failed to halt` | Driver open source ignore `NVreg_EnableGpuFirmware=0` sur RTX 30xx |
| 2 | `nvidia-driver-515` (Ubuntu 22.04) | `RmInitAdapter failed (0x23:0x65:1438)` | Driver trop ancien pour RTX 3060 LHR |
| 3 | `nvidia-driver-570` propriétaire sans param | `No devices were found` | Module chargé mais GSP firmware bloque |

### ✅ Solution finale qui fonctionne

```bash
# 1. Purger tous les drivers NVIDIA existants
sudo apt remove --purge -y '*nvidia*'
sudo apt autoremove -y
sudo apt clean

# 2. Installer le driver propriétaire 570 (PAS le -open)
sudo apt install -y nvidia-driver-570

# 3. Désactiver le GSP firmware (clé du succès sous ESXi passthrough)
echo 'options nvidia NVreg_EnableGpuFirmware=0' | sudo tee /etc/modprobe.d/nvidia.conf

# 4. Mettre à jour l'initramfs et rebooter
sudo update-initramfs -u
sudo reboot
```

> **Pourquoi `NVreg_EnableGpuFirmware=0` ?**  
> Sous ESXi en passthrough, le firmware GSP de NVIDIA tente de communiquer avec le matériel via des canaux qui ne sont pas disponibles dans un contexte d'hyperviseur. Ce paramètre désactive le GSP et force le driver à utiliser le mode RM (Resource Manager) classique, compatible avec le passthrough.  
> Le driver **propriétaire** (non `-open`) est le seul qui respecte ce paramètre sur les RTX 30xx.

### Vérification

```bash
nvidia-smi
```

Résultat attendu :
```
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 570.211.01    Driver Version: 570.211.01    CUDA Version: 12.8              |
| GPU  0  NVIDIA GeForce RTX 3060   ...  | 00000000:02:02.0 Off |                  N/A   |
|  0%   37C    P8    9W / 170W  |    0MiB / 12288MiB  |   0%  Default                   |
+-----------------------------------------------------------------------------------------+
```

---

## Étape 4 — Installer Ollama

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

### Configurer Ollama pour écouter sur le réseau

```bash
sudo systemctl edit ollama
```

Ajouter dans le fichier :
```ini
[Service]
Environment="OLLAMA_HOST=0.0.0.0"
```

```bash
sudo systemctl daemon-reload
sudo systemctl restart ollama
```

### Monter les modèles depuis le Synology (NFS)

```bash
# Créer le point de montage
sudo mkdir -p /mnt/ollama-models

# Ajouter dans /etc/fstab
echo "192.168.x.x:/volume1/ollama-models  /mnt/ollama-models  nfs  defaults,_netdev  0  0" | sudo tee -a /etc/fstab
sudo mount -a
```

Configurer Ollama pour utiliser ce chemin :

```bash
sudo systemctl edit ollama
# Ajouter :
# Environment="OLLAMA_MODELS=/mnt/ollama-models"
sudo systemctl restart ollama
```

### Puller un modèle

```bash
ollama pull gemma4:e4b   # 9.6 GB — tient entièrement dans les 12GB VRAM RTX 3060
```

---

## Étape 5 — Vérifier l'utilisation GPU

```bash
# Pendant une inférence — terminal 1
ollama run gemma4:e4b "Explique le modèle OSI en 3 phrases"

# Terminal 2 — observer le GPU
watch -n1 nvidia-smi
# ou
nvtop
```

### Résultats observés

| Modèle | VRAM utilisée | Tokens/s | Notes |
|---|---|---|---|
| `gemma4:e4b` | ~9.6 GB / 12 GB | ~50-80 t/s | Tient entièrement en VRAM ✅ |
| `gemma4:26b` | 11.4 GB + 8.6 GB RAM | ~14 t/s | Déborde en RAM → bottleneck PCIe |
| `gemma4:26b` (Mac M5 Pro) | mémoire unifiée | ~44 t/s | Plus rapide grâce à l'UMA |

> **Recommandation :** utiliser `gemma4:e4b` sur la VM pour les workflows n8n (rapide, 100% GPU).  
> Utiliser `gemma4:26b` sur le Mac M5 Pro pour les traitements lourds (OCR, génération longue).

---

## Notes supplémentaires

### Autostart de la VM au boot ESXi

```
ESXi UI → VM → Edit Settings → VM Options → Boot Options → Start automatically
```

> ⚠️ L'autostart ESXi Free peut être capricieux. Un bouton dans Home Assistant (via SSH + `vim-cmd`) est plus fiable.

### VMID de la VM ollama-gpu

```bash
vim-cmd vmsvc/getallvms | grep ollama
# ollama-gpu → VMID 17
```

### Commandes vim-cmd utiles

```bash
vim-cmd vmsvc/power.on 17       # démarrer
vim-cmd vmsvc/power.shutdown 17 # arrêt propre
vim-cmd vmsvc/power.getstate 17 # état actuel
```

---

## Références

- [Ollama documentation](https://ollama.com)
- [VMware ESXi PCI Passthrough documentation](https://docs.vmware.com/en/VMware-vSphere/8.0/vsphere-vm-administration/GUID-5B8D6B56-DEC7-4D43-AE1D-B80B4A87E25D.html)
- NVIDIA RTX 3060 (GA106) — architecture Ampere, 12GB GDDR6, CUDA 12.8
