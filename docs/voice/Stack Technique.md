# Voice — Stack Technique

## Vue d'ensemble

| Composant | Technologie | Rôle |
|-----------|------------|------|
| Whisper | `openai-whisper` | STT local |
| Groq | API REST (LPU) | STT cloud |
| edge-tts | `edge-tts` (Microsoft) | TTS |
| pydub | `pydub` + `ffmpeg` | Conversion audio |
| Celery | Queue `voice` | Exécution async |

## Tâches Celery

| Task | Queue | Retries | Détails |
|------|-------|---------|---------|
| `process_stt` | voice | 2 (10s delay) | Voir [stt/Stack Technique](stt/Stack%20Technique.md) |
| `process_tts` | voice | 2 (10s delay) | Voir [tts/Stack Technique](tts/Stack%20Technique.md) |

## Docker

Dockerfile dédié (`Dockerfile.voice`) car dépendances lourdes (~1.5 GB) :
- `ffmpeg`, `openai-whisper` + PyTorch, `spacy`

Concurrency : 1 worker (CPU/GPU-intensif).

```bash
docker compose up voice-worker -d
```

## Fichiers clés

```
ia/workers/orpheam_workers/voice/
├── tasks.py    # process_stt + process_tts
├── stt.py      # transcribe_audio() (lazy-loaded)
└── tts.py      # synthesize_speech() (lazy-loaded)
```
