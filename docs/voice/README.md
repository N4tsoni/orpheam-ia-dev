# Voice — STT & TTS

Le domaine Voice gère la **transcription audio** (Speech-to-Text) et la **synthèse vocale** (Text-to-Speech) pour l'assistant Jarvis.

## Comment ca marche

```
Audio → STT (Whisper ou Groq) → Texte
Texte → TTS (edge-tts) → Audio
```

Les deux tâches sont des jobs Celery sur la queue `voice`, déclenchés par Laravel via RabbitMQ.

## Docs du dossier

- [Stack technique](Stack%20technique.md) — Technos, config, Docker
- [Whisper](Whisper.md) — STT local (OpenAI Whisper)
- [Groq STT](Groq%20STT.md) — STT cloud rapide
- [edge-tts](edge-tts.md) — Synthèse vocale Microsoft
