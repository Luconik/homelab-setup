<div align="center">

![Luconik Banner](assets/Logo_Luconik.png)

# homelab-setup

**AI workstation · Docker · K3s · Traefik · n8n · Network labs**

[![GitHub](https://img.shields.io/badge/GitHub-Luconik-181717?style=flat-square&logo=github)](https://github.com/Luconik)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-nicolasculetto-0077B5?style=flat-square&logo=linkedin)](https://www.linkedin.com/in/nicolasculetto/)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-Server-E95420?style=flat-square&logo=ubuntu)](https://ubuntu.com/server)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=flat-square&logo=docker)](https://docs.docker.com/compose/)
[![K3s](https://img.shields.io/badge/K3s-Kubernetes-FFC61C?style=flat-square&logo=k3s)](https://k3s.io/)
[![Traefik](https://img.shields.io/badge/Traefik-Let's_Encrypt-24A1C1?style=flat-square&logo=traefikproxy)](https://traefik.io/traefik/)
[![n8n](https://img.shields.io/badge/n8n-Automation-EA4B71?style=flat-square&logo=n8n)](https://n8n.io/)

> 🇫🇷 [Français](#fr) | 🇬🇧 [English](#en)

</div>

---

<a name="fr"></a>
## 🇫🇷 Français

### Présentation

Ce dépôt documente mon homelab personnel actuel : une workstation IA Ubuntu toujours disponible, un petit cluster K3s, une station Windows orientée NetDevOps et plusieurs laboratoires réseau Aruba et Juniper.

L'infrastructure active n'utilise plus Proxmox, ESXi ni Nginx Proxy Manager. Les anciens guides sont conservés dans une section **Archives techniques** car ils restent utiles comme retours d'expérience, mais ils ne représentent plus l'architecture en production.

### Infrastructure actuelle

| Nœud | Plateforme | Ressources principales | Rôle |
|---|---|---|---|
| **ialbator** | Ubuntu 26.04 | Ryzen AI Max+ 395, 32 Go RAM | IA locale, KVM, Docker, Hermès, Home Assistant et automatisation |
| **k3s-master** | Ubuntu 24.04 | Intel Core i3-6100U, 32 Go RAM | Control plane K3s et Gitea |
| **k3s-worker1** | Ubuntu 24.04 | Intel Core i7-10710U, 64 Go RAM | Worker K3s |
| **Bakastation** | Windows 11 Pro | Intel Core i9-10850K, 128 Go RAM, RTX 3060 12 Go | Containerlab, Ansible et Terraform |
| **Bakasyno** | Synology DSM | NAS | Stockage, sauvegardes et synchronisation Obsidian |

### Architecture des services

```text
Internet
└── Traefik + Let's Encrypt
    └── Services publiés explicitement en HTTPS

ialbator — Ubuntu, toujours disponible
├── Hermes Agent — gateway Discord/Telegram et connecteurs MCP
├── Docker
│   ├── Traefik — reverse proxy et certificats ACME
│   ├── n8n — automatisation et bot Discord
│   ├── AdGuard Home — DNS local
│   └── serveurs MCP — intégrations isolées
├── PostgreSQL — données applicatives et n8n
├── Home Assistant — VM KVM
└── Synology Drive — synchronisation du vault Obsidian

Cluster K3s
├── k3s-master — control plane + Gitea
└── k3s-worker1 — workloads de découverte et GitOps

Bakastation — Windows 11
└── Containerlab + Ansible + Terraform
```

### Réseau et laboratoires

- Infrastructure domestique : gateway, switches et points d'accès UniFi.
- Lab Aruba : switches AOS-CX, points d'accès AOS 10 et HPE Aruba Networking Central.
- Lab Juniper : switches EX, points d'accès Juniper et Mist.
- Containerlab est exécuté depuis Bakastation pour les scénarios NetDevOps.

### Structure du dépôt

```text
homelab-setup/
├── agents/
│   └── hermes/              # Hermès Agent sur Ubuntu
├── docker/
│   └── n8n/                 # n8n, PostgreSQL et bot Discord
├── eve-ng/
│   ├── aos-cx-ova/          # Guide HPE NSP et image AOS-CX
│   ├── esxi/                # Archive : EVE-NG sur ESXi
│   └── proxmox/             # Archive : EVE-NG sur Proxmox
├── esxi/
│   └── ollama-gpu/          # Archive : passthrough RTX 3060
├── proxmox/
│   └── gpu-passthrough-rtx3060.md  # Archive
├── ubuntu-server/           # Archive : ancienne VM automation
└── assets/
```

### Guides actifs

| Guide | Description | Statut |
|---|---|---|
| [`agents/hermes/`](agents/hermes/) | Installation native, gateway et connecteurs MCP | ✅ Actuel |
| [`docker/n8n/`](docker/n8n/) | n8n durci, PostgreSQL, Discord et Nyaa | ✅ Actuel |
| [`eve-ng/aos-cx-ova/`](eve-ng/aos-cx-ova/) | Accès HPE NSP et préparation d'une image AOS-CX | ✅ Référence |

### Archives techniques

| Guide | Ancien environnement | Statut |
|---|---|---|
| [`eve-ng/esxi/`](eve-ng/esxi/) | EVE-NG sous VMware ESXi | 🗃️ Historique |
| [`eve-ng/proxmox/`](eve-ng/proxmox/) | EVE-NG sous Proxmox VE | 🗃️ Historique |
| [`esxi/ollama-gpu/`](esxi/ollama-gpu/) | Ollama et RTX 3060 en passthrough ESXi | 🗃️ Historique |
| [`proxmox/gpu-passthrough-rtx3060.md`](proxmox/gpu-passthrough-rtx3060.md) | RTX 3060 en passthrough Proxmox | 🗃️ Historique |
| [`ubuntu-server/`](ubuntu-server/) | VM `automation` sous Proxmox | 🗃️ Historique |

> Les commandes et versions des archives peuvent être obsolètes. Elles doivent être validées avec la documentation officielle avant réutilisation.

### Principes d'exploitation

- Traefik est l'unique reverse proxy de la plateforme active.
- Les certificats TLS sont gérés automatiquement avec Let's Encrypt/ACME.
- Seuls les services nécessaires sont publiés ; les bases de données restent sur les réseaux internes.
- Les images Docker importantes sont épinglées par version et, après validation, par digest.
- Les secrets restent dans des fichiers d'environnement protégés ou des secret stores, jamais dans Git.
- Les changements destructifs sont précédés d'un export ou d'une sauvegarde vérifiée.

### Dépôts liés

| Dépôt | Description |
|---|---|
| [`netdevops`](https://github.com/Luconik/netdevops) | Ansible et Terraform pour les laboratoires réseau |
| [`hpe-aruba-guides`](https://github.com/Luconik/hpe-aruba-guides) | Guides HPE Aruba Networking, NAC et intégrations |

---

<a name="en"></a>
## 🇬🇧 English

### Overview

This repository documents my current personal homelab: an always-on Ubuntu AI workstation, a small K3s cluster, a Windows NetDevOps workstation, and separate Aruba and Juniper network labs.

The active infrastructure no longer uses Proxmox, ESXi, or Nginx Proxy Manager. Previous guides remain available under **Technical archives** because they are still useful as field notes, but they no longer describe the production architecture.

### Current infrastructure

| Node | Platform | Main resources | Role |
|---|---|---|---|
| **ialbator** | Ubuntu 26.04 | Ryzen AI Max+ 395, 32 GB RAM | Local AI, KVM, Docker, Hermes, Home Assistant and automation |
| **k3s-master** | Ubuntu 24.04 | Intel Core i3-6100U, 32 GB RAM | K3s control plane and Gitea |
| **k3s-worker1** | Ubuntu 24.04 | Intel Core i7-10710U, 64 GB RAM | K3s worker |
| **Bakastation** | Windows 11 Pro | Intel Core i9-10850K, 128 GB RAM, RTX 3060 12 GB | Containerlab, Ansible and Terraform |
| **Bakasyno** | Synology DSM | NAS | Storage, backups and Obsidian synchronization |

### Service architecture

```text
Internet
└── Traefik + Let's Encrypt
    └── Explicitly published HTTPS services

ialbator — always-on Ubuntu server
├── Hermes Agent — Discord/Telegram gateway and MCP connectors
├── Docker
│   ├── Traefik — reverse proxy and ACME certificates
│   ├── n8n — automation and Discord bot
│   ├── AdGuard Home — local DNS
│   └── MCP servers — isolated integrations
├── PostgreSQL — application and n8n data
├── Home Assistant — KVM virtual machine
└── Synology Drive — Obsidian vault synchronization

K3s cluster
├── k3s-master — control plane + Gitea
└── k3s-worker1 — discovery and GitOps workloads

Bakastation — Windows 11
└── Containerlab + Ansible + Terraform
```

### Network and labs

- Home network: UniFi gateway, switches, and access points.
- Aruba lab: AOS-CX switches, AOS 10 access points, and HPE Aruba Networking Central.
- Juniper lab: EX switches, Juniper access points, and Mist.
- Containerlab runs on Bakastation for NetDevOps scenarios.

### Repository structure

```text
homelab-setup/
├── agents/
│   └── hermes/              # Hermes Agent on Ubuntu
├── docker/
│   └── n8n/                 # n8n, PostgreSQL and Discord bot
├── eve-ng/
│   ├── aos-cx-ova/          # HPE NSP and AOS-CX image guide
│   ├── esxi/                # Archive: EVE-NG on ESXi
│   └── proxmox/             # Archive: EVE-NG on Proxmox
├── esxi/
│   └── ollama-gpu/          # Archive: RTX 3060 passthrough
├── proxmox/
│   └── gpu-passthrough-rtx3060.md  # Archive
├── ubuntu-server/           # Archive: former automation VM
└── assets/
```

### Active guides

| Guide | Description | Status |
|---|---|---|
| [`agents/hermes/`](agents/hermes/) | Native installation, gateway, and MCP connectors | ✅ Current |
| [`docker/n8n/`](docker/n8n/) | Hardened n8n, PostgreSQL, Discord, and Nyaa | ✅ Current |
| [`eve-ng/aos-cx-ova/`](eve-ng/aos-cx-ova/) | HPE NSP access and AOS-CX image preparation | ✅ Reference |

### Technical archives

| Guide | Previous environment | Status |
|---|---|---|
| [`eve-ng/esxi/`](eve-ng/esxi/) | EVE-NG on VMware ESXi | 🗃️ Historical |
| [`eve-ng/proxmox/`](eve-ng/proxmox/) | EVE-NG on Proxmox VE | 🗃️ Historical |
| [`esxi/ollama-gpu/`](esxi/ollama-gpu/) | Ollama and RTX 3060 passthrough on ESXi | 🗃️ Historical |
| [`proxmox/gpu-passthrough-rtx3060.md`](proxmox/gpu-passthrough-rtx3060.md) | RTX 3060 passthrough on Proxmox | 🗃️ Historical |
| [`ubuntu-server/`](ubuntu-server/) | Former Proxmox `automation` VM | 🗃️ Historical |

> Commands and versions in archived guides may be outdated. Check current official documentation before reusing them.

### Operating principles

- Traefik is the only reverse proxy in the active platform.
- TLS certificates are managed automatically through Let's Encrypt/ACME.
- Only required services are published; databases remain on internal networks.
- Important Docker images are pinned by version and, after validation, by digest.
- Secrets stay in protected environment files or secret stores and are never committed.
- Destructive changes are preceded by a verified export or backup.

### Related repositories

| Repository | Description |
|---|---|
| [`netdevops`](https://github.com/Luconik/netdevops) | Ansible and Terraform for network labs |
| [`hpe-aruba-guides`](https://github.com/Luconik/hpe-aruba-guides) | HPE Aruba Networking, NAC, and integration guides |

---

*Last updated: July 2026 — [@Luconik](https://github.com/Luconik)*
