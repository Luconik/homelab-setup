<div align="center">

![Luconik Banner](assets/Logo_Luconik.png)

# homelab-setup

**Nicolas Culetto — Pre-Sales Systems Engineer @ HPE Aruba Networking**

*Infrastructure as Code · Self-hosted · NetDevOps · Network & Security*

[![LinkedIn](https://img.shields.io/badge/LinkedIn-nicolasculetto-0077B5?style=flat-square&logo=linkedin)](https://www.linkedin.com/in/nicolasculetto/)
[![GitHub](https://img.shields.io/badge/GitHub-Luconik-181717?style=flat-square&logo=github)](https://github.com/Luconik)
[![Proxmox](https://img.shields.io/badge/Hyperviseur-Proxmox_VE-E57000?style=flat-square&logo=proxmox)](https://www.proxmox.com/)
[![Docker](https://img.shields.io/badge/Conteneurs-Docker-2496ED?style=flat-square&logo=docker)](https://www.docker.com/)
[![n8n](https://img.shields.io/badge/Automatisation-n8n-EA4B71?style=flat-square&logo=n8n)](https://n8n.io/)
[![EVE-NG](https://img.shields.io/badge/Emulation-EVE--NG-FF6600?style=flat-square)](https://www.eve-ng.net/)

---

> 🇫🇷 Documentation disponible en français — English summary included in each section.  
> 🇬🇧 Documentation written in French — English summary available in each section.

</div>

---

## À propos / About

Ce dépôt documente l'infrastructure complète de mon homelab personnel, construit autour d'un **Intel NUC sous Proxmox VE**.

L'objectif est double :
- **Personnel** : avoir une infra reproductible, documentée et versionnée
- **Communautaire** : partager des guides terrain en français, notamment autour de l'écosystème **HPE Aruba Networking** et des outils DevOps/SecOps

> **EN** — This repository documents my personal homelab infrastructure built on a Proxmox VE Intel NUC. It covers virtualization, network emulation (EVE-NG), Docker-based automation with n8n, Discord bots, NAS integration (Synology), and security hardening. All guides are written in French with English summaries.

---

## 🖥 Stack technique / Tech Stack

| Composant | Technologie | Rôle |
|-----------|-------------|------|
| Hyperviseur | Proxmox VE (Intel NUC) | Virtualisation de l'ensemble |
| Émulation réseau | EVE-NG | Labs HPE Aruba AOS-CX / Juniper vJunos |
| Reverse proxy | Nginx Proxy Manager | Exposition services + SSL Let's Encrypt |
| DNS / WAF | Cloudflare | Protection périmétrique externe |
| Firewall host | UFW | Règles d'accès locales |
| Automatisation | n8n (Docker) | Workflows, notifications, bots |
| Bot Discord | Culetto-Home-Bot | Interface Discord → n8n workflows |
| Base de données | PostgreSQL (Docker) | Backend n8n |
| NAS | Synology | Stockage + client torrent (qBittorrent) |
| GitLab CE | gitlab.culetto.fr | Dépôt Git + CI/CD pipelines |
| Automation VM | automation.culetto.fr | Runner GitLab + Ansible + Terraform |

---

## 📁 Structure du repo

```
homelab-setup/
├── assets/                        # Bannières et images
├── eve-ng/
│   ├── esxi/                      # Guide installation EVE-NG sur ESXi
│   └── proxmox/                   # Guide installation EVE-NG sur Proxmox VE
├── ubuntu-server/                 # Guide installation Ubuntu Server (VM Proxmox)
├── docker/
│   ├── n8n/                       # Stack n8n + PostgreSQL (Docker Compose)
│   └── nginx-proxy-manager/       # Config NPM + Cloudflare SSL
└── security/
    ├── ufw/                       # Règles UFW recommandées
    └── crowdsec/                  # Intégration CrowdSec (WIP)
```

---

## 🤖 Automatisation n8n + Discord

L'une des parties les plus actives du homelab : un stack **n8n + PostgreSQL + Discord Bot** qui centralise les notifications et les actions manuelles.

### Architecture

```
Discord (Culetto-Home-Bot)
        │
        ▼
   n8n (Docker)  ◄──► PostgreSQL
        │
        ├──► qBittorrent (Synology NAS — 192.168.0.113)
        ├──► Nyaa.si RSS (notifications nouveaux torrents)
        └──► Proxmox API (scheduler VMs)
```

### Workflows principaux

| Workflow | Déclencheur | Description |
|----------|-------------|-------------|
| `!dl <url>` | Discord | Téléchargement manuel de torrent via qBittorrent |
| `!skip` | Discord | Ignorer une notification torrent en attente |
| `!status` | Discord | Statut des téléchargements actifs |
| `!newseason` | Discord | Déclarer une nouvelle saison anime (purge auto) |
| Nyaa RSS Notify | Scheduler | Surveillance RSS Nyaa.si → notification Discord |
| Seasonal Purge | Scheduler | Nettoyage automatique en fin de saison |

> 📂 Guide complet et Docker Compose → [`docker/n8n/`](docker/n8n/)

---

## 🔐 Central NAC + Intune — HPE Aruba TechDocs

Guide technique publié sur le portail officiel **HPE Aruba TechDocs** :  
**Central NAC with Microsoft Intune UEM Onboarding**

> 📖 [Lire la documentation sur HPE Aruba TechDocs](https://arubanetworking.hpe.com/techdocs/NAC/central-nac/central-nac-uem-onboarding-intune/)

Ce guide couvre l'intégration complète **Aruba Central NAC + Microsoft Intune** pour le déploiement de certificats SCEP en 802.1X — un cas d'usage enterprise fréquent et peu documenté en français.

**Technologies couvertes :**
- HPE Aruba Central NAC (Policy Manager)
- Microsoft Intune (UEM) + SCEP certificate profile
- 802.1X authentication (EAP-TLS)
- ClearPass Policy Manager

---

## 🗺 Architecture globale

```
Internet
   │
   ▼
Cloudflare WAF / DNS
   │
   ▼
Nginx Proxy Manager (VM Proxmox)
   │
   ├──► gitlab.culetto.fr       ← GitLab CE
   ├──► automation.culetto.fr   ← Ansible + Terraform + GitLab Runner
   ├──► n8n.culetto.fr          ← n8n automation
   └──► pve1.culetto.fr         ← Proxmox VE (accès restreint)
   
Synology NAS (192.168.0.113)
   └──► qBittorrent :8080       ← Géré par n8n workflows
```

---

## 📚 Guides disponibles

| Section | Contenu | Statut |
|---------|---------|--------|
| [EVE-NG / ESXi](eve-ng/esxi/) | Installation EVE-NG sur VMware ESXi | ✅ Disponible |
| [EVE-NG / Proxmox](eve-ng/proxmox/) | Installation EVE-NG sur Proxmox VE | ✅ Disponible |
| [Ubuntu Server](ubuntu-server/) | Installation VM Ubuntu Server sur Proxmox | 🚧 En cours |
| [Docker / n8n](docker/n8n/) | Stack n8n + PostgreSQL + Discord Bot | 🚧 En cours |
| [Docker / NPM](docker/nginx-proxy-manager/) | Nginx Proxy Manager + SSL Cloudflare | 🚧 En cours |
| [Security / UFW](security/ufw/) | Règles UFW pour homelab | 🚧 En cours |
| [Security / CrowdSec](security/crowdsec/) | Intégration CrowdSec | 📋 Planifié |
| [GitLab CE](docs/04-gitlab/) | Installation GitLab CE sur Proxmox | 📋 Planifié |

---

## 🔗 Repos associés

| Repo | Description |
|------|-------------|
| [aruba-netdevops](https://github.com/Luconik/aruba-netdevops) | Automatisation réseau HPE Aruba AOS-CX avec Ansible, Terraform et GitLab CI/CD |

---

## 📬 Contact

- **LinkedIn** : [linkedin.com/in/nicolasculetto](https://www.linkedin.com/in/nicolasculetto/)
- **Email** : nicolas@culetto.fr
- **GitHub** : [github.com/Luconik](https://github.com/Luconik)

---

> Les guides sont librement réutilisables avec mention de l'auteur (CC BY 4.0).

<div align="center">

*Made with ❤️ and too much coffee — Nicolas Culetto / Luconik*

[![LinkedIn](https://img.shields.io/badge/LinkedIn-nicolasculetto-0077B5?style=flat-square&logo=linkedin)](https://www.linkedin.com/in/nicolasculetto/)

</div>
