# Hermes Agent on Ubuntu / Hermès Agent sur Ubuntu

> 🇫🇷 [Français](#fr) | 🇬🇧 [English](#en)

This guide documents a secure, always-on Hermes Agent deployment on an Ubuntu homelab server. Tokens, user IDs, channel IDs and private addresses are deliberately omitted.

---

<a name="fr"></a>
## Français

### Architecture retenue

Hermès est installé **nativement** pour conserver un accès simple au terminal local. Les dépendances MCP sont isolées dans des conteneurs lorsque cela apporte une frontière de sécurité utile.

```text
Ubuntu Server
├── Hermes Agent — installation native et environnement Python isolé
├── Gateway — service systemd utilisateur, Discord et Telegram
├── Home Assistant — API HTTPS, token dans l'environnement
└── MCP
    ├── Obsidian filesystem — vault RO + répertoire assistant RW
    └── n8n — outils de consultation sélectionnés
```

### Installation et configuration

Installer Hermès en suivant la procédure officielle du projet, puis lancer l'assistant :

```bash
hermes setup
```

Choix utilisés dans ce déploiement :

- fournisseur OpenAI via Codex OAuth ;
- modèle principal `gpt-5.6-sol` ;
- backend terminal local ;
- navigateur Chromium local ;
- recherche DuckDuckGo, avec SearXNG auto-hébergé prévu ensuite.

Les versions de modèles et de paquets évoluent : les valider avant chaque nouvelle installation et éviter les tags d'image flottants en production.

### Gateway Discord et Telegram

```bash
hermes setup gateway
hermes gateway start
hermes gateway status
journalctl --user -u hermes-gateway -f
```

Bonnes pratiques :

- utiliser des bots dédiés ;
- configurer une allowlist d'utilisateurs ;
- ne jamais stocker les tokens dans Git ;
- activer le service systemd utilisateur et le linger pour survivre à la déconnexion ;
- désactiver la création automatique de threads Discord si les réponses doivent rester dans le canal : `discord.auto_thread=false`.

### Home Assistant

Stocker l'URL et le jeton longue durée dans le fichier d'environnement privé d'Hermès, avec des permissions restrictives :

```bash
chmod 600 ~/.hermes/.env
```

Commencer par un test en lecture. Ne donner accès aux commandes à effet qu'après avoir défini les entités autorisées et les confirmations nécessaires.

### Obsidian avec MCP filesystem

Le montage recommandé donne accès à toute la documentation en lecture, mais limite l'écriture au répertoire réservé à l'assistant :

```yaml
volumes:
  - /path/to/vault:/vault:ro
  - /path/to/vault/System/Assistant:/vault/System/Assistant:rw
```

Épingler l'image MCP par digest, puis vérifier depuis Hermès qu'une note peut être lue et qu'une écriture hors de `System/Assistant` est refusée.

### n8n avec MCP

Installer le serveur depuis un catalogue de confiance, l'épingler sur une version ou un commit connu et n'activer par défaut que les outils de consultation. Les opérations qui modifient ou suppriment des workflows doivent demander une confirmation explicite.

### Checklist de sécurité

- [ ] Secrets uniquement dans un fichier privé ou un secret store
- [ ] Discord et Telegram protégés par allowlist
- [ ] Images et dépendances épinglées
- [ ] Vault Obsidian en lecture seule hors espace assistant
- [ ] Outils MCP réduits au minimum nécessaire
- [ ] Logs gateway vérifiés après chaque changement
- [ ] Sauvegarde de configuration testée, sans secrets

---

<a name="en"></a>
## English

### Design

Hermes runs **natively** for straightforward access to the local terminal. MCP dependencies run in containers when isolation provides a useful security boundary.

```text
Ubuntu Server
├── Hermes Agent — native install and isolated Python environment
├── Gateway — user systemd service, Discord and Telegram
├── Home Assistant — HTTPS API, token stored in the environment
└── MCP
    ├── Obsidian filesystem — read-only vault + writable assistant directory
    └── n8n — selected read-oriented tools
```

Run `hermes setup`, select the required provider, local terminal backend and browser, then configure messaging with `hermes setup gateway`.

Security baseline:

- use dedicated bots and user allowlists;
- never commit tokens, user IDs or channel IDs;
- enable the user systemd service and linger for an always-on gateway;
- use `discord.auto_thread=false` when replies must remain in the source channel;
- mount the Obsidian vault read-only and expose one narrowly scoped writable directory;
- pin MCP images and servers to immutable versions;
- expose read-only n8n tools first and require confirmation for mutations;
- validate Home Assistant in read-only mode before enabling device control.

Useful operations:

```bash
hermes gateway status
hermes gateway restart
journalctl --user -u hermes-gateway -f
```

---

*Validated in a homelab deployment in July 2026. Review current upstream documentation before installing.*
