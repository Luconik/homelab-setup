# Ubuntu Server 24.04 LTS — Installation & Configuration
# Ubuntu Server 24.04 LTS — Installation & Configuration

> 🇫🇷 [Français](#fr) | 🇬🇧 [English](#en)

---

<a name="fr"></a>
## 🇫🇷 Français

### Objectif

Ce guide couvre l'installation et la configuration d'**Ubuntu Server 24.04 LTS** sur une VM Proxmox, ainsi que le provisioning complet de la VM `automation` — utilisée pour Ansible, Terraform, Docker et le GitLab Runner.

---

## Partie 1 — Installation Ubuntu Server 24.04 LTS

### 1.1 Télécharger l'ISO

```bash
# Sur Proxmox, télécharger l'ISO dans le storage local
# Interface web Proxmox → local → ISO Images → Download from URL

https://releases.ubuntu.com/24.04/ubuntu-24.04.2-live-server-amd64.iso
```

### 1.2 Créer la VM dans Proxmox

Dans l'interface web Proxmox :

| Paramètre | Valeur recommandée (VM automation) |
|-----------|-----------------------------------|
| Name | `automation` |
| OS | Linux 6.x (Ubuntu) |
| CPU | 2 vCPU (type : host) |
| RAM | 4 GB (ballooning activé) |
| Disk | 40 GB (VirtIO SCSI, thin provisioning) |
| Network | VirtIO, bridge `vmbr0` |
| ISO | `ubuntu-24.04.2-live-server-amd64.iso` |

### 1.3 Installation — étapes clés

1. **Language** → French (ou English)
2. **Keyboard layout** → French (ou QWERTY selon préférence)
3. **Type d'installation** → Ubuntu Server (minimized)
4. **Réseau** → Configurer l'IP fixe ici (voir section 2.1) ou laisser DHCP et fixer après
5. **Storage** → Use entire disk → LVM (pas ZFS pour une VM)
6. **Profile** :
   - Your name : `Nicolas Culetto`
   - Server name : `automation`
   - Username : `luconik`
   - Password : (mot de passe fort)
7. **SSH** → ✅ Install OpenSSH server → importer la clé depuis GitHub (`github.com/Luconik`)
8. **Snaps** → Ne rien sélectionner → Done

---

## Partie 2 — Configuration post-installation

### 2.1 IP fixe (Netplan)

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

```yaml
network:
  version: 2
  ethernets:
    ens18:                          # Adapter selon le nom de l'interface (ip a)
      dhcp4: false
      addresses:
        - 192.168.x.x/24           # IP fixe souhaitée
      routes:
        - to: default
          via: 192.168.x.1         # Gateway
      nameservers:
        addresses:
          - 1.1.1.1
          - 8.8.8.8
```

```bash
sudo netplan apply
ip a                               # Vérifier l'IP
```

### 2.2 SSH — configuration sécurisée

```bash
sudo nano /etc/ssh/sshd_config
```

```
PermitRootLogin no
PasswordAuthentication no          # Après avoir importé la clé SSH
PubkeyAuthentication yes
```

```bash
sudo systemctl restart ssh
```

### 2.3 Mises à jour système

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget git unzip vim htop net-tools
```

### 2.4 QEMU Guest Agent (Proxmox)

```bash
sudo apt install -y qemu-guest-agent
sudo systemctl enable --now qemu-guest-agent
```

---

## Partie 3 — Installation Ansible

### 3.1 Créer le virtualenv dédié

```bash
sudo apt install -y python3-venv python3-pip

# Créer le venv pour les labs AOS-CX Aruba
sudo mkdir -p /opt/ansible-aruba
sudo chown $USER:$USER /opt/ansible-aruba
python3 -m venv /opt/ansible-aruba/venv
```

### 3.2 Installer ansible-core 2.17

```bash
source /opt/ansible-aruba/venv/bin/activate

# Installer ansible-core version spécifique
pip install --upgrade pip
pip install ansible-core==2.17.*

# Vérifier
ansible --version
# ansible [core 2.17.x]
```

### 3.3 Installer la collection AOS-CX

```bash
# Toujours dans le venv
ansible-galaxy collection install arubanetworks.aoscx

# Vérifier
ansible-galaxy collection list | grep aoscx
# arubanetworks.aoscx  x.x.x
```

### 3.4 Activer le venv automatiquement (optionnel)

```bash
echo 'source /opt/ansible-aruba/venv/bin/activate' >> ~/.bashrc
source ~/.bashrc
```

### 3.5 Vérification complète

```bash
ansible --version
python3 -c "import ansible; print(ansible.__version__)"
ansible-galaxy collection list | grep aoscx
```

---

## Partie 4 — Installation Terraform

### 4.1 Installer Terraform 1.5.x

```bash
# Méthode officielle HashiCorp (apt)
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt update

# Installer la version 1.5.x spécifique
sudo apt install -y terraform=1.5.*

# Vérifier
terraform version
# Terraform v1.5.x
```

### 4.2 Empêcher les mises à jour automatiques (version pinning)

```bash
sudo apt-mark hold terraform
```

### 4.3 Initialiser le provider AOS-CX

```bash
cd ~/netdevops/terraform/
terraform init

# Vérifier le provider
terraform providers
# provider[registry.terraform.io/aruba/aoscx]
```

---

## Partie 5 — Installation Docker & Docker Compose

### 5.1 Installer Docker Engine

```bash
# Désinstaller les anciennes versions
sudo apt remove -y docker docker-engine docker.io containerd runc

# Installer les dépendances
sudo apt install -y ca-certificates curl gnupg lsb-release

# Ajouter le repo officiel Docker
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
sudo tee /etc/apt/sources.list.d/docker.list

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### 5.2 Configurer Docker pour l'utilisateur courant

```bash
sudo usermod -aG docker $USER
newgrp docker

# Vérifier
docker --version
docker compose version
```

### 5.3 Démarrer Docker au boot

```bash
sudo systemctl enable --now docker
```

---

## Partie 6 — Installation GitLab Runner

### 6.1 Installer le runner

```bash
# Ajouter le repo GitLab
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | sudo bash

# Installer
sudo apt install -y gitlab-runner
```

### 6.2 Enregistrer le runner

```bash
sudo gitlab-runner register \
  --url https://gitlab.your-domain.com \
  --token <RUNNER_TOKEN>           \
  --executor shell                 \
  --description "automation-runner" \
  --tag-list "shell,ansible,terraform"
```

> 💡 Le token se récupère dans GitLab → Settings → CI/CD → Runners → New project runner.

### 6.3 Vérifier le runner

```bash
sudo gitlab-runner status
sudo gitlab-runner list
```

### 6.4 Configurer le runner pour utiliser le venv Ansible

Éditer `/etc/gitlab-runner/config.toml` pour que le runner utilise le bon venv :

```toml
[[runners]]
  name = "automation-runner"
  url = "https://gitlab.your-domain.com"
  executor = "shell"
  [runners.custom_build_dir]
  [runners.cache]
  environment = ["ANSIBLE_COLLECTIONS_PATH=/opt/ansible-aruba/collections",
                 "ANSIBLE_PYTHON_INTERPRETER=/opt/ansible-aruba/venv/bin/python3"]
```

```bash
sudo systemctl restart gitlab-runner
```

---

## Récapitulatif — Versions installées

| Composant | Version | Commande de vérification |
|-----------|---------|--------------------------|
| Ubuntu Server | 24.04 LTS | `lsb_release -a` |
| Python | 3.12.x | `python3 --version` |
| ansible-core | 2.17.x | `ansible --version` |
| arubanetworks.aoscx | dernière | `ansible-galaxy collection list \| grep aoscx` |
| Terraform | 1.5.x | `terraform version` |
| Docker Engine | dernière stable | `docker --version` |
| Docker Compose | dernière stable | `docker compose version` |
| GitLab Runner | dernière stable | `gitlab-runner --version` |

---

## Références

- [Ubuntu Server 24.04 LTS](https://ubuntu.com/download/server)
- [Ansible Installation Guide](https://docs.ansible.com/ansible/latest/installation_guide/)
- [Terraform Install — Ubuntu](https://developer.hashicorp.com/terraform/install#linux)
- [Docker Engine — Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
- [GitLab Runner — Installation](https://docs.gitlab.com/runner/install/linux-repository.html)
- [`../docker/n8n/`](../docker/n8n/) — Stack n8n + PostgreSQL sur cette VM
- 🔗 [`netdevops/ansible/`](https://github.com/Luconik/netdevops/tree/main/ansible) — Labs AOS-CX (Ansible installé sur cette VM)
- 🔗 [`netdevops/terraform/`](https://github.com/Luconik/netdevops/tree/main/terraform) — Terraform (installé sur cette VM)
- 🔗 [`netdevops/docs/gitlab-cicd/`](https://github.com/Luconik/netdevops/tree/main/docs/gitlab-cicd) — Pipeline GitLab CI/CD (runner sur cette VM)

---
---

<a name="en"></a>
## 🇬🇧 English

### Purpose

This guide covers the installation and configuration of **Ubuntu Server 24.04 LTS** on a Proxmox VM, and the full provisioning of the `automation` VM — used for Ansible, Terraform, Docker, and the GitLab Runner.

---

## Part 1 — Ubuntu Server 24.04 LTS Installation

### VM specs (automation)

| Parameter | Value |
|-----------|-------|
| Name | `automation` |
| CPU | 2 vCPU (type: host) |
| RAM | 4 GB |
| Disk | 40 GB (VirtIO SCSI, thin) |
| Network | VirtIO, bridge `vmbr0` |

### Key installation steps

1. Type → Ubuntu Server (minimized)
2. Storage → Use entire disk → LVM
3. Profile → username: `luconik`, server: `automation`
4. SSH → ✅ Install OpenSSH → import key from `github.com/Luconik`
5. Snaps → nothing → Done

---

## Part 2 — Post-install configuration

### Static IP (Netplan)

```yaml
# /etc/netplan/00-installer-config.yaml
network:
  version: 2
  ethernets:
    ens18:
      dhcp4: false
      addresses: [192.168.x.x/24]
      routes:
        - to: default
          via: 192.168.x.1
      nameservers:
        addresses: [1.1.1.1, 8.8.8.8]
```

```bash
sudo netplan apply
```

---

## Part 3 — Ansible installation

```bash
# Create dedicated venv
sudo mkdir -p /opt/ansible-aruba
python3 -m venv /opt/ansible-aruba/venv
source /opt/ansible-aruba/venv/bin/activate

# Install ansible-core 2.17
pip install ansible-core==2.17.*
ansible --version    # ansible [core 2.17.x]

# Install AOS-CX collection
ansible-galaxy collection install arubanetworks.aoscx
ansible-galaxy collection list | grep aoscx
```

---

## Part 4 — Terraform installation

```bash
# Add HashiCorp repo
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update

# Install Terraform 1.5.x
sudo apt install -y terraform=1.5.*
sudo apt-mark hold terraform    # Pin version
terraform version               # Terraform v1.5.x
```

---

## Part 5 — Docker & Docker Compose

```bash
# Install Docker Engine (official repo)
sudo apt install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list
sudo apt update && sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Add user to docker group
sudo usermod -aG docker $USER && newgrp docker
sudo systemctl enable --now docker
```

---

## Part 6 — GitLab Runner

```bash
# Install
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | sudo bash
sudo apt install -y gitlab-runner

# Register
sudo gitlab-runner register \
  --url https://gitlab.your-domain.com \
  --token <RUNNER_TOKEN> \
  --executor shell \
  --description "automation-runner" \
  --tag-list "shell,ansible,terraform"

# Verify
sudo gitlab-runner status
```

---

## Installed versions summary

| Component | Version | Check command |
|-----------|---------|---------------|
| Ubuntu Server | 24.04 LTS | `lsb_release -a` |
| Python | 3.12.x | `python3 --version` |
| ansible-core | 2.17.x | `ansible --version` |
| arubanetworks.aoscx | latest | `ansible-galaxy collection list \| grep aoscx` |
| Terraform | 1.5.x | `terraform version` |
| Docker Engine | latest stable | `docker --version` |
| Docker Compose | latest stable | `docker compose version` |
| GitLab Runner | latest stable | `gitlab-runner --version` |

---

## References

- [Ubuntu Server 24.04 LTS](https://ubuntu.com/download/server)
- [Ansible Installation Guide](https://docs.ansible.com/ansible/latest/installation_guide/)
- [Terraform Install — Ubuntu](https://developer.hashicorp.com/terraform/install#linux)
- [Docker Engine — Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
- [GitLab Runner Install](https://docs.gitlab.com/runner/install/linux-repository.html)
- 🔗 [`netdevops/ansible/`](https://github.com/Luconik/netdevops/tree/main/ansible) — AOS-CX labs (Ansible installed on this VM)
- 🔗 [`netdevops/terraform/`](https://github.com/Luconik/netdevops/tree/main/terraform) — Terraform (installed on this VM)
- 🔗 [`netdevops/docs/gitlab-cicd/`](https://github.com/Luconik/netdevops/tree/main/docs/gitlab-cicd) — GitLab CI/CD pipeline (runner on this VM)

---

*Last updated: March 2026 — [@Luconik](https://github.com/Luconik)*
