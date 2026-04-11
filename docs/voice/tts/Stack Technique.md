# TTS — Stack Technique

## Technos

| Composant | Technologie | Rôle |
|-----------|------------|------|
| edge-tts | `edge-tts` (Microsoft Edge Read Aloud) | Synthèse vocale neuronale |
| Celery | Queue `voice` | Exécution async |

## Task Celery

| Task | Queue | Retries | Delay |
|------|-------|---------|-------|
| `process_tts` | voice | 2 | 10s |

## Caractéristiques edge-tts

- **Gratuit** — utilise l'API Microsoft Edge, pas d'authentification
- **300+ voix** dans 40+ langues
- **Voix neuronales** — qualité naturelle
- **Léger** — pas de GPU, pas de modèle local
- **Limites** — pas de fine-tuning, pas de clonage de voix, dépendance réseau

## Docker

Inclus dans `Dockerfile.voice`. Pas de dépendance lourde côté TTS.
