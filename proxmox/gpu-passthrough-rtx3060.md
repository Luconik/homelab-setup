# GPU Passthrough — RTX 3060 sur Proxmox VE

> Testé sur : ASUS ROG STRIX Z390-F Gaming / i9-10850K / Proxmox VE 6.17 / RTX 3060 12GB  
> Guest : Ubuntu Server 24.04 LTS + Ollama

---

## Prérequis BIOS

- **Intel VT-x** : activé
- **Intel VT-d** : activé
- **Above 4G Decoding** : activé (recommandé pour les GPU modernes)
- **CSM** : désactivé

---

## 1. Vérifier l'IOMMU

Depuis le shell Proxmox :

```bash
dmesg | grep -e DMAR -e IOMMU
```

Résultat attendu : présence de `Intel(R) Virtualization Technology for Directed I/O`.

Vérifier que le GPU est isolé dans son propre groupe IOMMU :

```bash
find /sys/kernel/iommu_groups/ -type l | sort -V
```

La RTX 3060 doit apparaître seule dans son groupe (ex: groupe 2) :

```
/sys/kernel/iommu_groups/2/devices/0000:01:00.0   ← GPU
/sys/kernel/iommu_groups/2/devices/0000:01:00.1   ← Audio HDMI
```

---

## 2. Récupérer les IDs PCI du GPU

```bash
lspci -n -s 01:00
```

Exemple de résultat :

```
01:00.0 0300: 10de:2504 (rev a1)   ← RTX 3060 GPU
01:00.1 0403: 10de:228e (rev a1)   ← RTX 3060 Audio HDMI
```

---

## 3. Configurer vfio-pci

```bash
# Lier le GPU au driver vfio-pci
echo "options vfio-pci ids=10de:2504,10de:228e disable_vga=1" > /etc/modprobe.d/vfio.conf

# Blacklister les drivers nvidia et nouveau
echo "blacklist nouveau
blacklist nvidia
blacklist nvidiafb" >> /etc/modprobe.d/blacklist.conf

# Forcer le chargement de vfio avant nvidia/nouveau
echo "softdep nouveau pre: vfio-pci
softdep nvidia pre: vfio-pci" >> /etc/modprobe.d/blacklist.conf

# Ajouter les modules vfio au démarrage
echo "vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd" >> /etc/modules

# Appliquer
update-initramfs -u -k all
reboot
```

Après reboot, vérifier que vfio a bien pris le contrôle :

```bash
lspci -k -s 01:00
```

Résultat attendu :

```
01:00.0 VGA compatible controller: NVIDIA Corporation GA106 [GeForce RTX 3060 ...]
        Kernel driver in use: vfio-pci
01:00.1 Audio device: NVIDIA Corporation GA106 High Definition Audio Controller
        Kernel driver in use: vfio-pci
```

---

## 4. Créer la VM

Paramètres essentiels :

| Paramètre | Valeur |
|-----------|--------|
| Machine | **q35** |
| BIOS | **OVMF (UEFI)** |
| EFI Disk | 4MB (créé automatiquement) |
| Display | **VirtIO** (ne pas cocher Primary GPU pendant l'install) |
| OS | Ubuntu Server 24.04 LTS |

```bash
# Appliquer machine q35 + OVMF sur une VM existante (VMID=100)
qm set 100 --machine q35
qm set 100 --bios ovmf
qm set 100 --efidisk0 local-lvm:1,format=raw
qm set 100 --vga virtio
```

---

## 5. Ajouter le PCI Device

Dans l'interface Proxmox : **VM → Hardware → Add → PCI Device**

- Type : **Raw Device**
- Device : `0000:01:00.0` (RTX 3060)
- ✅ **All Functions** (inclut le `01:00.1` audio automatiquement)
- ✅ **PCI-Express**
- ☐ Primary GPU (laisser décoché pendant l'install Ubuntu)

---

## 6. Installer les drivers Nvidia dans la VM

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y linux-headers-$(uname -r)
sudo apt install -y nvidia-driver-570 nvidia-utils-570
sudo reboot
```

Vérifier après reboot :

```bash
nvidia-smi
```

Résultat attendu :

```
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 570.211.01    Driver Version: 570.211.01    CUDA Version: 12.8              |
|  GPU  Name           | Bus-Id          | Memory-Usage        | GPU-Util               |
|  0  NVIDIA GeForce RTX 3060 | 00000000:06:10.0 | 1MiB / 12288MiB | 0%             |
+-----------------------------------------------------------------------------------------+
```

---

## 7. Installer Ollama

```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama pull gemma4:e4b
```

Tester avec monitoring GPU en parallèle :

```bash
# Terminal 1
watch -n 1 nvidia-smi

# Terminal 2
ollama run gemma4:e4b "Test GPU passthrough"
```

Performances observées avec `gemma4:e4b` :

| Métrique | Valeur |
|----------|--------|
| VRAM utilisée | ~9.8 GB / 12 GB |
| GPU-Util (inférence) | ~80% |
| Vitesse génération | ~75 tokens/sec |
| PCIe | GEN 3x16 |

---

## Notes

- Sur ESXi, le passthrough échouait avec des erreurs `NVRM GSP firmware errors` — Proxmox n'a pas ce problème avec le driver 570.
- Le paramètre `disable_vga=1` dans `vfio.conf` est important pour éviter les conflits au boot.
- Ne pas cocher **Primary GPU** tant que les drivers ne sont pas installés dans la VM (perte d'accès VNC sinon).
- Les modèles Ollama peuvent être stockés sur NFS (Synology) via `/mnt/ollama-models/` pour économiser l'espace disque VM.
