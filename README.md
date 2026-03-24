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

> 🇫🇷 Documentation principale en français — un résumé en anglais est disponible dans chaque section.  
> 🇬🇧 Main documentation in French — an English summary is available in each section.

</div>

---

## À propos

Ce dépôt documente l'infrastructure complète de mon homelab personnel, construit autour d'un **Intel NUC sous Proxmox VE**.

L'objectif est double :
- **Personnel** : avoir une infrastructure reproductible, documentée et versionnée
- **Communautaire** : partager des guides terrain en français, notamment autour de l'écosystème **HPE Aruba Networking** et des outils DevOps/SecOps

> **EN** — This repository documents my personal homelab infrastructure built on a Proxmox VE Intel NUC. It covers virtualization, network emulation (EVE-NG), Docker-based automation with n8n, Discord bots, NAS integration (Synology), and security hardening. All guides are written in French with English summaries.

---

## Stack technique

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

> **EN** — The homelab runs on a single Intel NUC with Proxmox VE as the hypervisor. Key services include EVE-NG for network emulation, n8n for workflow automation, a Discord bot for remote control, a Synology NAS for storage, and a self-hosted GitLab instance for CI/CD.

---

## Structure du dépôt

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

> **EN** — The repository is organized by technology area: EVE-NG network emulation guides, Ubuntu Server setup, Docker-based services (n8n automation stack), and security hardening (UFW, CrowdSec).

---

## Automatisation n8n + Discord

L'une des parties les plus actives du homelab : un stack **n8n + PostgreSQL + Discord Bot** qui centralise les notifications et les actions à distance.

### Architecture

```
Discord (Culetto-Home-Bot)
        │
        ▼
   n8n (Docker)  ◄──► PostgreSQL
        │
        ├──► qBittorrent (Synology NAS)
        ├──► Nyaa.si RSS (notifications nouveaux torrents)
        └──► Proxmox API (scheduler VMs)
```

### Commandes Discord disponibles

| Commande | Description |
|----------|-------------|
| `!dl <url>` | Téléchargement manuel de torrent via qBittorrent |
| `!skip` | Ignorer une notification torrent en attente |
| `!status` | Statut des téléchargements actifs |
| `!newseason` | Déclarer une nouvelle saison (purge automatique) |

> **EN** — The n8n automation stack connects a Discord bot to qBittorrent (running on a Synology NAS), an RSS feed watcher (Nyaa.si), and the Proxmox API. Discord commands allow remote control of downloads and scheduled tasks without direct server access.

📂 Guide complet et Docker Compose → [`docker/n8n/`](docker/n8n/)

---

## Central NAC + Intune — HPE Aruba TechDocs

Guide technique publié sur le portail officiel **HPE Aruba TechDocs** :

> 📖 [Central NAC with Microsoft Intune UEM Onboarding](https://arubanetworking.hpe.com/techdocs/NAC/central-nac/central-nac-uem-onboarding-intune/)

Ce guide couvre l'intégration complète **Aruba Central NAC + Microsoft Intune** pour le déploiement de certificats SCEP en 802.1X — un cas d'usage enterprise fréquent et peu documenté.

Technologies couvertes : HPE Aruba Central NAC · Microsoft Intune (UEM) · SCEP · 802.1X EAP-TLS · ClearPass Policy Manager

> **EN** — Technical guide published on the official HPE Aruba TechDocs portal. Covers the full integration of Aruba Central NAC with Microsoft Intune for SCEP certificate-based 802.1X authentication — a common enterprise use case that is rarely documented end-to-end.

---

## Architecture globale

```
Internet
   │
   ▼
Cloudflare WAF / DNS
   │
   ▼
Nginx Proxy Manager (VM Proxmox)
   │
   ├──► gitlab.culetto.fr         ← GitLab CE
   ├──► automation.culetto.fr     ← Ansible + Terraform + GitLab Runner
   ├──► n8n.culetto.fr            ← n8n automation
   └──► pve1.culetto.fr           ← Proxmox VE (accès restreint)

Synology NAS (réseau local)
   └──► qBittorrent               ← Géré par n8n workflows
```

> **EN** — All services are exposed through Nginx Proxy Manager behind Cloudflare WAF. The Synology NAS hosts qBittorrent and is controlled remotely via n8n workflows triggered from Discord.

---

## Guides disponibles

| Section | Contenu | Statut |
|---------|---------|--------|
| [EVE-NG / ESXi](eve-ng/esxi/) | Installation EVE-NG sur VMware ESXi | ✅ Disponible |
| [EVE-NG / Proxmox](eve-ng/proxmox/) | Installation EVE-NG sur Proxmox VE | ✅ Disponible |
| [Ubuntu Server](ubuntu-server/) | Installation VM Ubuntu Server sur Proxmox | 🚧 En cours |
| [Docker / n8n](docker/n8n/) | Stack n8n + PostgreSQL + Discord Bot | 🚧 En cours |
| [Docker / NPM](docker/nginx-proxy-manager/) | Nginx Proxy Manager + SSL Cloudflare | 🚧 En cours |
| [Security / UFW](security/ufw/) | Règles UFW pour homelab | 🚧 En cours |
| [Security / CrowdSec](security/crowdsec/) | Intégration CrowdSec | 📋 Planifié |

---

## Repo associé

| Repo | Description |
|------|-------------|
| [netdevops](https://github.com/Luconik/netdevops) | Automatisation réseau HPE Aruba AOS-CX — Ansible, Terraform, GitLab CI/CD |

---

## Contact

- 🔗 **LinkedIn** : [linkedin.com/in/nicolasculetto](https://www.linkedin.com/in/nicolasculetto/)
- 📧 **Email** : nicolas@culetto.fr
- 🐙 **GitHub** : [github.com/Luconik](https://github.com/Luconik)

---

> Les guides sont librement réutilisables avec mention de l'auteur (CC BY 4.0).

<div align="center">

*Made with ❤️ and too much coffee — Nicolas Culetto / Luconik*

[![LinkedIn](https://img.shields.io/badge/LinkedIn-nicolasculetto-0077B5?style=flat-square&logo=linkedin)](https://www.linkedin.com/in/nicolasculetto/)

</div>
