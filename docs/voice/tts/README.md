# TTS — Text-to-Speech

Synthèse vocale texte → audio pour l'assistant Jarvis.

## Providers

| Provider | Type | Qualité | Coût | GPU requis |
|----------|------|---------|------|------------|
| [edge-tts](edge-tts.md) | Cloud (Microsoft Edge) | Bonne (voix neuronales) | Gratuit | Non |

## Flux

```
Texte → TTS Provider → Audio (mp3)
```

Task Celery : `process_tts` sur la queue `voice`.

## Config

```env
TTS_PROVIDER=edge
TTS_VOICE=fr-FR-DeniseNeural
```

## Voix disponibles (francais)

| Voix | Genre |
|------|-------|
| `fr-FR-DeniseNeural` | Femme (défaut) |
| `fr-FR-HenriNeural` | Homme |

## Fichiers

```
ia/workers/orpheam_workers/voice/
├── tts.py      # synthesize_speech() (lazy-loaded)
└── tasks.py    # process_tts
```
