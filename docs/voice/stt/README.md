# STT — Speech-to-Text

Transcription audio → texte pour l'assistant Jarvis.

## Providers

| Provider | Type | Latence | Coût | GPU requis |
|----------|------|---------|------|------------|
| [Whisper](Whisper.md) | Local (OpenAI) | 5-30s selon modèle | Gratuit | Recommandé |
| [Groq STT](Groq%20STT.md) | Cloud (LPU) | 1-2s | Pay-per-use | Non |

## Flux

```
Audio (wav/mp3) → STT Provider → Texte brut
```

Task Celery : `process_stt` sur la queue `voice`.

## Config

```env
STT_PROVIDER=groq          # groq | whisper
STT_MODEL=base              # tiny | base | small | medium | large-v3
GROQ_API_KEY=gsk_...        # requis si provider=groq
```

## Fichiers

```
ia/workers/orpheam_workers/voice/
├── stt.py      # transcribe_audio() (lazy-loaded)
└── tasks.py    # process_stt
```
