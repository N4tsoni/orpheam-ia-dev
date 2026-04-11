# Whisper — STT Local

## C'est quoi ?

[Whisper](https://github.com/openai/whisper) est un modèle de transcription audio open-source par OpenAI. Il tourne **en local** sur le worker, pas besoin d'API externe.

## Modèles disponibles

| Modèle | Params | VRAM | Vitesse | Qualité |
|--------|--------|------|---------|---------|
| `tiny` | 39M | ~1 GB | Tres rapide | Basse |
| `base` | 74M | ~1 GB | Rapide | Correcte |
| `small` | 244M | ~2 GB | Moyen | Bonne |
| `medium` | 769M | ~5 GB | Lent | Tres bonne |
| `large-v3` | 1.5B | ~10 GB | Tres lent | Excellente |

Par défaut on utilise `base` — bon compromis vitesse/qualité.

## Langues

Whisper est multilingue. On utilise principalement `fr` (francais) et `en` (anglais). La détection de langue est automatique si non spécifiée.

## Quand l'utiliser ?

- Pas de connexion internet / pas de clé API
- Volume important de transcriptions (pas de coût par requete)
- GPU disponible sur le worker
- Latence acceptable (5-30s selon modèle)

## Config

```env
STT_PROVIDER=whisper
STT_MODEL=base
```
