# Voice — Stack technique

## Technos

| Composant | Technologie | Rôle |
|-----------|------------|------|
| Whisper | `openai-whisper` | STT local |
| Groq | API REST | STT cloud (Whisper optimisé) |
| edge-tts | `edge-tts` (Microsoft) | TTS |
| pydub | `pydub` + `ffmpeg` | Conversion audio |
| Celery | Queue `voice` | Exécution async |

## Tâches Celery

| Task | Queue | Retries |
|------|-------|---------|
| `process_stt` | voice | 2 (10s delay) |
| `process_tts` | voice | 2 (10s delay) |

## Configuration

```env
STT_PROVIDER=groq          # groq | whisper
STT_MODEL=base              # tiny | base | small | medium | large-v3
GROQ_API_KEY=gsk_...        # requis si provider=groq
TTS_PROVIDER=edge
TTS_VOICE=fr-FR-DeniseNeural
```

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
