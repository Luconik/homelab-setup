<div align="center">

![Luconik Banner](assets/Logo_Luconik.png)

# homelab-setup

**Infrastructure homelab — Proxmox · EVE-NG · Docker · GitLab · Security**

[![GitHub](https://img.shields.io/badge/GitHub-Luconik-181717?style=flat-square&logo=github)](https://github.com/Luconik)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-nicolasculetto-0077B5?style=flat-square&logo=linkedin)](https://www.linkedin.com/in/nicolasculetto/)
[![Proxmox](https://img.shields.io/badge/Proxmox-VE-E57000?style=flat-square&logo=proxmox)](https://www.proxmox.com)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=flat-square&logo=docker)](https://docs.docker.com/compose/)
[![n8n](https://img.shields.io/badge/n8n-automation-EA4B71?style=flat-square)](https://n8n.io)

> 🇫🇷 [Français](#fr) | 🇬🇧 [English](#en)

</div>

---

<a name="fr"></a>
## 🇫🇷 Français

### Présentation

Ce repo documente l'infrastructure de mon homelab personnel, basé sur un **Intel NUC** sous **Proxmox VE**. Il couvre l'installation et la configuration de l'ensemble des services : virtualisation, émulation réseau, conteneurs Docker, sécurité et automatisation.

---

### Hardware

| Composant | Détail |
|-----------|--------|
| **Host** | Intel NUC |
| **Hyperviseur** | Proxmox VE (`pve1.your-domain.com`) |
| **NAS** | Synology (`192.168.x.x`) |
| **Réseau** | Aruba APs + switch managé |

---

### Structure du repo

```
homelab-setup/
├── eve-ng/
│   ├── esxi/                    ← EVE-NG sur ESXi
│   ├── proxmox/                 ← EVE-NG sur Proxmox
│   └── aos-cx-ova/              ← Compte HPE NSP + OVA AOS-CX
│       └── screenshots/
├── gitlab/                      ← Installation GitLab CE
├── ubuntu-server/               ← Configuration Ubuntu Server
├── docker/
│   ├── n8n/                     ← n8n + PostgreSQL + Discord Bot
│   └── nginx-proxy-manager/     ← Reverse proxy + SSL
├── security/
│   └── ufw/                     ← Firewall UFW
└── assets/
    └── Logo_Luconik.png
```

---

### Guides disponibles

| Dossier | Description | Statut |
|---------|-------------|--------|
| [`eve-ng/esxi/`](eve-ng/esxi/) | Installation EVE-NG sur ESXi | ✅ |
| [`eve-ng/proxmox/`](eve-ng/proxmox/) | Installation EVE-NG sur Proxmox | ✅ |
| [`eve-ng/aos-cx-ova/`](eve-ng/aos-cx-ova/) | Compte HPE NSP + téléchargement OVA AOS-CX | ✅ |
| [`gitlab/`](gitlab/) | Installation GitLab CE sur Proxmox | ✅ |
| [`ubuntu-server/`](ubuntu-server/) | Configuration Ubuntu Server | ✅ |
| [`docker/n8n/`](docker/n8n/) | n8n + PostgreSQL + Discord Bot (`Culetto-Home-Bot`) | ✅ |
| [`docker/nginx-proxy-manager/`](docker/nginx-proxy-manager/) | Reverse proxy Nginx + SSL Let's Encrypt | ✅ |
| [`security/ufw/`](security/ufw/) | Configuration firewall UFW | 🔜 |

---

### Stack Docker

```
automation.your-domain.com (Ubuntu Server VM sur Proxmox)
│
├── n8n                 → n8n.your-domain.com (HTTPS via NPM)
├── PostgreSQL          → Base de données n8n (n8n_postgres)
└── Nginx Proxy Manager → Reverse proxy pour tous les services
```

**Workflows n8n actifs :**

| Workflow | Description |
|----------|-------------|
| `Nyaa Notify` | Notifications RSS anime/manga VOSTFR → Discord |
| `!dl` | Téléchargement torrent via qBittorrent (NAS Synology) |
| `!skip` | Ignore une entrée RSS en attente |
| `!status` | Statut des téléchargements en cours |
| `!newseason` | Réinitialisation suivi saisonnier |

---

### Sécurité

| Couche | Solution |
|--------|----------|
| WAF | Cloudflare (frontal) |
| Reverse proxy | Nginx Proxy Manager + Let's Encrypt |
| Firewall host | UFW |
| Firewall hyperviseur | Proxmox firewall |

---

### Repos liés

| Repo | Description |
|------|-------------|
| [`netdevops`](https://github.com/Luconik/netdevops) | Ansible + Terraform AOS-CX + pipeline GitLab CI/CD |
| [`hpe-aruba-guides`](https://github.com/Luconik/hpe-aruba-guides) | GreenLake SSO, NAC + Intune, Central workspace |

---
---

<a name="en"></a>
## 🇬🇧 English

### Overview

This repo documents my personal homelab infrastructure, based on an **Intel NUC** running **Proxmox VE**. It covers installation and configuration of all services: virtualization, network emulation, Docker containers, security, and automation.

---

### Repo structure

```
homelab-setup/
├── eve-ng/
│   ├── esxi/                    ← EVE-NG on ESXi
│   ├── proxmox/                 ← EVE-NG on Proxmox
│   └── aos-cx-ova/              ← HPE NSP account + AOS-CX OVA download
├── gitlab/                      ← GitLab CE installation
├── ubuntu-server/               ← Ubuntu Server setup
├── docker/
│   ├── n8n/                     ← n8n + PostgreSQL + Discord Bot
│   └── nginx-proxy-manager/     ← Reverse proxy + SSL
├── security/
│   └── ufw/                     ← UFW firewall
└── assets/
    └── Logo_Luconik.png
```

---

### Available guides

| Folder | Description | Status |
|--------|-------------|--------|
| [`eve-ng/esxi/`](eve-ng/esxi/) | EVE-NG on ESXi | ✅ |
| [`eve-ng/proxmox/`](eve-ng/proxmox/) | EVE-NG on Proxmox | ✅ |
| [`eve-ng/aos-cx-ova/`](eve-ng/aos-cx-ova/) | HPE NSP account + AOS-CX OVA download | ✅ |
| [`gitlab/`](gitlab/) | GitLab CE on Proxmox | ✅ |
| [`ubuntu-server/`](ubuntu-server/) | Ubuntu Server setup | ✅ |
| [`docker/n8n/`](docker/n8n/) | n8n + PostgreSQL + Discord Bot | ✅ |
| [`docker/nginx-proxy-manager/`](docker/nginx-proxy-manager/) | Nginx reverse proxy + SSL | ✅ |
| [`security/ufw/`](security/ufw/) | UFW firewall | 🔜 |

---

### Related repos

| Repo | Description |
|------|-------------|
| [`netdevops`](https://github.com/Luconik/netdevops) | Ansible + Terraform AOS-CX + GitLab CI/CD pipeline |
| [`hpe-aruba-guides`](https://github.com/Luconik/hpe-aruba-guides) | GreenLake SSO, NAC + Intune, Central workspace |

---

*Last updated: March 2026 — [@Luconik](https://github.com/Luconik)*
