# edge-tts — Synthèse Vocale

## C'est quoi ?

[edge-tts](https://github.com/rany2/edge-tts) est une lib Python qui utilise le service **Microsoft Edge Read Aloud** pour la synthèse vocale. C'est gratuit et la qualité est bonne.

## Voix francaises

| Voix | Genre | Style |
|------|-------|-------|
| `fr-FR-DeniseNeural` | Femme | Naturelle, neutre |
| `fr-FR-HenriNeural` | Homme | Naturel, neutre |

Par défaut : `fr-FR-DeniseNeural`

## Avantages

- **Gratuit** (utilise l'API Edge, pas d'auth)
- **Bonne qualité** — voix neuronales naturelles
- **Multilingue** — 300+ voix dans 40+ langues
- **Simple** — pas de setup, pas de GPU

## Inconvénients

- Dépendance réseau (API Microsoft)
- Pas de fine-tuning possible
- Pas de clonage de voix

## Config

```env
TTS_PROVIDER=edge
TTS_VOICE=fr-FR-DeniseNeural
```
