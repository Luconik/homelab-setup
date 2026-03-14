<div align="center">

![Luconik Banner](assets/Logo_Luconik.png)

# homelab-setup

**Nicolas Culetto — Pre-Sales Systems Engineer @ HPE Aruba Networking**

*Infrastructure as Code · Self-hosted · NetDevOps · Sécurité réseau*

[![LinkedIn](https://img.shields.io/badge/LinkedIn-nicolasculetto-0077B5?style=flat-square&logo=linkedin)](https://www.linkedin.com/in/nicolasculetto/)
[![GitHub](https://img.shields.io/badge/GitHub-Luconik-181717?style=flat-square&logo=github)](https://github.com/Luconik)
[![Proxmox](https://img.shields.io/badge/Hyperviseur-Proxmox_VE-E57000?style=flat-square&logo=proxmox)](https://www.proxmox.com/)
[![Docker](https://img.shields.io/badge/Conteneurs-Docker-2496ED?style=flat-square&logo=docker)](https://www.docker.com/)
[![n8n](https://img.shields.io/badge/Automatisation-n8n-EA4B71?style=flat-square&logo=n8n)](https://n8n.io/)

---

> 🇫🇷 Documentation disponible en français — English summary included in each section.  
> 🇬🇧 Documentation written in French — English summary available in each section.

</div>

---

## À propos / About

Ce dépôt documente l'infrastructure complète de mon homelab personnel, construit autour d'un **Intel NUC sous Proxmox VE**.

L'objectif est double :
- **Personnel** : avoir une infra reproductible, documentée et versionnée
- **Communautaire** : partager des guides terrain en français, notamment autour de l'écosystème **HPE Aruba Networking** et des outils DevOps/NetOps

> **EN** — This repository documents my personal homelab infrastructure built on Proxmox VE. It covers OS installation, EVE-NG network emulation, Docker-based automation with n8n, and security hardening. All guides are written in French with English summaries.

---

## Stack technique / Tech Stack

| Couche | Technologie |
|---|---|
| Hyperviseur | Proxmox VE (Intel NUC) |
| Réseau exposé | Nginx Proxy Manager + Cloudflare DNS + SSL Let's Encrypt |
| Stockage | Synology NAS |
| Conteneurs | Docker Compose |
| Automatisation | n8n + PostgreSQL + Discord Bot |
| Émulation réseau | EVE-NG Community Edition |
| Sécurité | UFW + CrowdSec |
| DNS / Reverse Proxy | culetto.fr (domaine perso) |

---

## Structure du dépôt / Repository Structure

```
homelab-setup/
├── assets/                  # Bannière et ressources visuelles
├── eve-ng/                  # Guides d'installation EVE-NG
│   ├── esxi/                #   → Sur VMware ESXi
│   └── proxmox/             #   → Sur Proxmox VE
├── ubuntu-server/           # Guide d'installation Ubuntu Server 22.04 LTS
├── docker/                  # Docker Compose & services self-hosted
│   ├── n8n/                 #   → Automatisation + Discord Bot + qBittorrent
│   └── nginx-proxy-manager/ #   → Reverse proxy + SSL
└── security/                # Durcissement sécurité
    ├── ufw/                 #   → Pare-feu UFW
    └── crowdsec/            #   → IPS collaboratif CrowdSec
```

---

## Guides disponibles / Available Guides

### 🖥️ [EVE-NG](./eve-ng/)

Déploiement d'EVE-NG Community Edition pour émulation réseau Aruba / Cisco / Juniper.

| Guide | Description |
|---|---|
| [EVE-NG sur ESXi](./eve-ng/esxi/README.md) | Installation d'EVE-NG comme VM sur VMware ESXi |
| [EVE-NG sur Proxmox](./eve-ng/proxmox/README.md) | Installation d'EVE-NG comme VM sur Proxmox VE |

> **EN** — Step-by-step guides to deploy EVE-NG for network emulation (Aruba AOS-CX, Cisco IOS, Juniper vJunos). Includes OVA import, nested virtualization setup, and node configuration.

---

### 🐧 [Ubuntu Server](./ubuntu-server/)

Installation et configuration de base d'Ubuntu Server 22.04 LTS sous Proxmox.

> **EN** — Ubuntu Server 22.04 LTS installation guide: VM creation in Proxmox, OS installation, SSH hardening, static IP, and initial system configuration.

---

### 🐳 [Docker & Automatisation](./docker/)

Services self-hosted déployés via Docker Compose sur la VM Ubuntu Server.

| Service | Description |
|---|---|
| [n8n](./docker/n8n/) | Orchestrateur de workflows + bot Discord + intégration qBittorrent |
| [Nginx Proxy Manager](./docker/nginx-proxy-manager/) | Reverse proxy avec SSL automatique via Cloudflare |

> **EN** — Self-hosted services deployed with Docker Compose: n8n workflow automation (Discord bot, scheduled tasks, PostgreSQL deduplication), and Nginx Proxy Manager with Cloudflare DNS challenge for automatic SSL.

---

### 🔒 [Sécurité / Security](./security/)

Durcissement de l'exposition des services homelab sur Internet.

| Composant | Description |
|---|---|
| [UFW](./security/ufw/) | Pare-feu Linux — règles et bonnes pratiques |
| [CrowdSec](./security/crowdsec/) | IPS collaboratif — détection et blocage d'intrusions |

> **EN** — Security hardening for homelab services: UFW firewall rules, CrowdSec collaborative IPS, and Nginx Proxy Manager security configuration.

---

## Schéma d'architecture / Architecture Overview

```
Internet
    │
    ▼
[Cloudflare DNS + Proxy]
    │  HTTPS / SSL Let's Encrypt
    ▼
[Nginx Proxy Manager] ──────── LXC Proxmox (10.224.100.21)
    │
    ├──▶ [n8n]          VM Ubuntu (10.224.100.12) — port 5678
    ├──▶ [GitLab CE]    VM Ubuntu (10.224.100.xx) — port 80/443
    └──▶ [Autres services self-hosted...]

[Synology NAS] ◀── qBittorrent (192.168.0.113:8080)
```

---

## Roadmap

- [x] EVE-NG sur ESXi
- [x] EVE-NG sur Proxmox
- [x] Ubuntu Server 22.04 LTS
- [x] Docker Compose + n8n + Discord Bot
- [x] Nginx Proxy Manager + Cloudflare SSL
- [x] UFW + CrowdSec
- [ ] Guide OVA AOS-CX sur EVE-NG
- [ ] Guide Juniper vJunos sur EVE-NG
- [ ] GitLab CE self-hosted
- [ ] Home Assistant integration

---

## Dépôts liés / Related Repositories

| Dépôt | Description |
|---|---|
| [aruba-netdevops](https://github.com/Luconik/aruba-netdevops) | Ansible + Terraform pour AOS-CX — GitLab CI/CD pipeline |

---

## Licence / License

Ce projet est sous licence [MIT](./LICENSE).  
Les guides sont librement réutilisables avec mention de l'auteur.

---

<div align="center">

*Made with ❤️ and too much coffee — Nicolas Culetto / Luconik*

[![LinkedIn](https://img.shields.io/badge/LinkedIn-nicolasculetto-0077B5?style=flat-square&logo=linkedin)](https://www.linkedin.com/in/nicolasculetto/)

</div>
