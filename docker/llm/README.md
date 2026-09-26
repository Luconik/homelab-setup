# Qwen3.8 27B Q8 sur ialbator

Runtime local validé sur Ryzen AI MAX+ 395 / Radeon 8060S, via Gufo et ROCm.

## État retenu

| Élément | Valeur |
|---|---|
| Modèle | `Qwen3.8-27B-UD-Q8_K_XL.gguf` |
| Brouillon spéculatif | `Qwen3.8-27B-DFlash2-Q4_K_M.gguf` |
| Contexte | 65 536 tokens |
| Service | `qwen3.8:27b` |
| API | OpenAI-compatible, `http://127.0.0.1:8089/v1` |
| Exposition | Loopback uniquement |
| Redémarrage | `unless-stopped` |

Le groupe numérique `990` est nécessaire pour accéder à `/dev/kfd` et à
`renderD128` depuis le conteneur. Les noms `render` et `video` du conteneur ne
sont pas suffisants sur cet hôte.

## Validation du 26 septembre 2026

- Raisonnement, streaming et tool-calling validés.
- Débit mesuré : environ 31 tok/s sur une réponse de 256 tokens.
- Charge mémoire : environ 39 Gio utilisés, avec ~21 Gio disponibles.

## Exploitation

Les poids sont attendus dans `/home/luconik/llm-models/qwen3.8-27b/` sur
ialbator et ne sont jamais inclus dans Git.

```bash
docker compose -f docker-compose.qwen3.8-q8.yml up -d
curl http://127.0.0.1:8089/health
```

Ollama reste installé comme solution de retour arrière, mais sans modèle gardé
chaud. Qwen3.8 Flash-Next 125B est conservé hors service : le réglage BIOS
`Auto` n'expose que 64 Gio à ROCm, insuffisant pour le charger avec Gufo.
