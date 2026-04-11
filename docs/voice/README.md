# Voice — STT & TTS

Le domaine Voice gère la **transcription audio** (Speech-to-Text) et la **synthèse vocale** (Text-to-Speech) pour l'assistant Jarvis.

## Comment ca marche

```
Audio → STT (Whisper ou Groq) → Texte
Texte → TTS (edge-tts) → Audio
```

Les deux tâches sont des jobs Celery sur la queue `voice`, déclenchés par Laravel via RabbitMQ.

## Sous-dossiers

- [stt/](stt/) — Speech-to-Text (Whisper local, Groq cloud)
- [tts/](tts/) — Text-to-Speech (edge-tts Microsoft)
