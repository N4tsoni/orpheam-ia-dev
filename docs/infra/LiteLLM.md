# LiteLLM — Proxy LLM Centralisé

## C'est quoi ?

[LiteLLM](https://docs.litellm.ai/) est un **proxy** qui expose une API compatible OpenAI (`/v1/chat/completions`) et route les requetes vers n'importe quel provider LLM (OpenAI, Anthropic, Google, etc.).

## Pourquoi ?

- **Un seul endpoint** pour tous les modèles → pas de code spécifique par provider
- **Changer de modèle** sans toucher au code (juste la config)
- **Rate limiting, cache, logging** centralisés
- **Une seule clé API** côté workers (la master key LiteLLM)

## Notre setup

```
Worker → LiteLLM (http://litellm:4000/v1) → OpenRouter → Provider final
```

Les workers appellent LiteLLM avec la master key. LiteLLM forward vers OpenRouter avec la vraie clé API.

## Modèles configurés

Définis dans `litellm_config.yaml` :

| Alias (côté code) | Provider réel | Rôle |
|-------|---------------|------|
| `google/gemini-2.5-flash` | OpenRouter | LLM principal (extraction KG) |
| `google/gemini-2.0-flash-lite-001` | OpenRouter | Small model (reranking) |
| `openai/text-embedding-3-small` | OpenRouter | Embeddings |
| `anthropic/claude-sonnet-4` | OpenRouter | Assistant / fallback |

## Configuration

```env
LLM_BASE_URL=http://litellm:4000/v1    # URL du proxy
LITELLM_MASTER_KEY=sk-litellm-orpheam   # Auth vers le proxy
OPENROUTER_API_KEY=sk-or-...             # Clé OpenRouter (dans le proxy)
```

## Dashboard

http://localhost:4000/ui — logs des requetes, coûts, latences.

## Docker

Service dans les deux docker-compose. Healthcheck sur `/health/readiness`.

```yaml
litellm:
  image: ghcr.io/berriai/litellm:main-latest
  ports: ["4000:4000"]
  volumes: ["./litellm_config.yaml:/app/config.yaml"]
```
