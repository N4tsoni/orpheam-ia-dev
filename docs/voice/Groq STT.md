# Groq STT — Transcription Cloud

## C'est quoi ?

[Groq](https://groq.com) propose une API de transcription basée sur Whisper mais exécutée sur leurs **LPU** (Language Processing Units) — du hardware spécialisé tres rapide.

## Avantages

- **Tres rapide** : ~1-2 secondes par requete
- **Meme qualité** que Whisper (c'est le meme modèle)
- **Pas de GPU** nécessaire sur le worker

## Inconvénients

- Nécessite une clé API (`GROQ_API_KEY`)
- Coût par requete (pay-per-use)
- Dépendance réseau

## Quand l'utiliser ?

- En production (rapidité)
- Pas de GPU disponible
- Volume modéré de transcriptions

## Config

```env
STT_PROVIDER=groq
GROQ_API_KEY=gsk_...
```
