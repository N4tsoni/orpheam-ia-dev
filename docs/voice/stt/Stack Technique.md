# STT — Stack Technique

## Technos

| Composant | Technologie | Rôle |
|-----------|------------|------|
| Whisper | `openai-whisper` | STT local, modèles tiny → large-v3 |
| Groq | API REST (LPU) | STT cloud, Whisper optimisé hardware |
| pydub | `pydub` + `ffmpeg` | Conversion / preprocessing audio |
| Celery | Queue `voice` | Exécution async |

## Task Celery

| Task | Queue | Retries | Delay |
|------|-------|---------|-------|
| `process_stt` | voice | 2 | 10s |

## Dépendances système

- `ffmpeg` — conversion audio (requis par pydub et whisper)
- `openai-whisper` + PyTorch — modèle local (optionnel si provider=groq)

## Docker

Inclus dans `Dockerfile.voice` (~1.5 GB avec PyTorch + Whisper).
Concurrency : 1 worker (CPU/GPU-intensif).
