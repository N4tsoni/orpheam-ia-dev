# Recap — Intégration LiteLLM & Graphiti (Sprint 1 + Sprint 2.1)

## Vue d'ensemble

Tous les appels LLM, embeddings et reranking passent désormais par un **proxy LiteLLM centralisé**, rendant la stack provider-agnostique. Graphiti (knowledge graph temporel) est connecté au proxy.

```
Application / Worker
    → OrpheamGraphitiClient (lib)
        → LiteLLM Proxy (http://litellm:4000/v1)
            → OpenRouter
                → Provider final (Google, OpenAI, Anthropic)
```

## Fichiers modifiés

### orpheam-libs (framework)

| Fichier                            | Changement                                                                                                    |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| `orpheam/llm/router.py`            | Ajout `api_key` + `custom_llm_provider="openai"`, suppression `fallbacks=` côté client                        |
| `orpheam/graph/graphiti_client.py` | Refactoré : plus de dépendance Gemini directe, `_ProxyCompatibleOpenAIClient` pour contourner `/v1/responses` |
| `orpheam/graph/search_config.py`   | **Nouveau** — `OrpheamSearchConfig` : config de recherche décomposée avec recettes pré-configurées            |
| `orpheam/graph/__init__.py`        | Export `OrpheamSearchConfig`                                                                                  |

### ia/workers (production)

| Fichier | Changement |
|---------|-----------|
| `docker-compose.yml` | Service `litellm`, healthcheck corrigé, env vars LiteLLM/Graphiti |
| `docker-compose.test.yml` | Service `litellm` standalone (dev/test) |
| `litellm_config.yaml` | 4 modèles via OpenRouter, cache local, `health_check_ephem_key: false` |
| `src/domain/kg_builder/services/neo4j_service.py` | Factory mise à jour : `LITELLM_MASTER_KEY`, `LLM_BASE_URL`, nouveaux noms de modèles |
| `tests/notebooks/01_litellm_proxy_test.ipynb` | Validation proxy LiteLLM (health, LLM, embeddings, OrpheamRouter) |
| `tests/notebooks/03_graphiti_litellm_test.ipynb` | Validation Graphiti via proxy (indices, ingestion, recherche, inspection Neo4j) |

## Architecture LiteLLM

### Flux des appels

```
OrpheamRouter.chat("query")
    → litellm.completion(custom_llm_provider="openai", api_key="sk-litellm-orpheam")
        → POST http://litellm:4000/v1/chat/completions
            → OpenRouter (OPENROUTER_API_KEY)
                → google/gemini-2.5-flash (ou autre modèle)
```

### Modèles configurés (`litellm_config.yaml`)

| Nom (côté client) | Provider réel | Rôle |
|-------------------|---------------|------|
| `google/gemini-2.5-flash` | OpenRouter | LLM principal (extraction KG, dedup, entity resolution) |
| `google/gemini-2.0-flash-lite-001` | OpenRouter | Small model (tâches simples, reranking) |
| `openai/text-embedding-3-small` | OpenRouter | Embeddings (vectorisation entités/relations) |
| `anthropic/claude-sonnet-4` | OpenRouter | LLM général (assistant, autres workers) |

### Configuration env vars

| Variable | Défaut (dev) | Rôle |
|----------|-------------|------|
| `LITELLM_MASTER_KEY` | `sk-litellm-orpheam` | Auth vers le proxy |
| `OPENROUTER_API_KEY` | (requis) | Clé OpenRouter (dans `.env`) |
| `LLM_BASE_URL` | `http://litellm:4000/v1` | URL du proxy |
| `GRAPHITI_LLM_MODEL` | `google/gemini-2.5-flash` | Modèle principal Graphiti |
| `GRAPHITI_SMALL_MODEL` | `google/gemini-2.0-flash-lite-001` | Small model Graphiti |
| `GRAPHITI_EMBEDDING_MODEL` | `openai/text-embedding-3-small` | Modèle embeddings |

## Architecture Graphiti

### Qu'est-ce que Graphiti ?

Graphiti (par Zep) est un moteur de **knowledge graph temporel**. Il transforme du texte brut en graphe de connaissances structuré dans Neo4j.

### Flux d'ingestion (`add_episode`)

```
Texte brut + reference_time
    → LLM : extraction d'entités et relations
    → Déduplication contre le graphe existant (entity resolution)
    → Gestion temporelle (expired_at sur les faits obsolètes)
    → Embedding des entités et relations
    → Persistance dans Neo4j (Entity, Episodic, RELATES_TO, MENTIONS)
```

### Schéma Neo4j

**Nodes :**
| Label | Rôle | Propriétés clés |
|-------|------|-----------------|
| `Entity` | Entités extraites | `name`, `summary`, `group_id`, `embedding` |
| `Episodic` | Texte source ingéré | `name`, `content`, `reference_time` |
| `Community` | Clusters d'entités | `name`, `summary` |

**Relations :**
| Type | Entre | Propriétés clés |
|------|-------|-----------------|
| `RELATES_TO` | Entity → Entity | `fact`, `valid_at`, `expired_at`, `embedding` |
| `MENTIONS` | Episodic → Entity | Lie épisode → entités générées |
| `HAS_MEMBER` | Community → Entity | Appartenance communauté |

### Temporalité

Graphiti ne supprime **jamais** de données. Quand un fait change :
- L'ancien fait reçoit un `expired_at` (date d'expiration)
- Le nouveau fait est créé avec un `valid_at`
- On peut requêter l'état du graphe à n'importe quelle date

### Flux de recherche (`search`)

```
Query en langage naturel
    → Embedding de la query (text-embedding-3-small)
    → Recherche parallèle :
        1. Vector search (cosine similarity sur embeddings)
        2. Full-text search (BM25 sur noms et faits)
        3. BFS optionnel (parcours du graphe)
    → Reranking (RRF, MMR, ou cross-encoder)
    → Résultats triés par pertinence
```

### OrpheamSearchConfig — Modes de recherche

```python
from orpheam.graph.search_config import OrpheamSearchConfig

# Recettes pré-configurées
config = OrpheamSearchConfig.hybrid_rrf()              # rapide, bon équilibre
config = OrpheamSearchConfig.hybrid_cross_encoder()     # précis (LLM reranking)
config = OrpheamSearchConfig.hybrid_mmr(mmr_lambda=0.7) # diversité des résultats
config = OrpheamSearchConfig.edges_only()               # faits/relations uniquement
config = OrpheamSearchConfig.nodes_only()               # entités uniquement

# Custom
config = OrpheamSearchConfig(
    search_edges=True,
    search_nodes=True,
    search_communities=False,
    methods=("bm25", "cosine", "bfs"),
    reranker="cross_encoder",
    limit=20,
)

# Utilisation
edges = await graphiti.search("query", group_id="mon_graphe")
results = await graphiti.search("query", group_id="mon_graphe", config=config)
```

**Méthodes de recherche :**
| Méthode | Description |
|---------|-------------|
| `bm25` | Mots-clés (full-text index Neo4j) |
| `cosine` | Sémantique (embeddings) |
| `bfs` | Parcours du graphe (connexions indirectes) |

**Stratégies de reranking :**
| Reranker | Description | Coût |
|----------|-------------|------|
| `rrf` | Reciprocal Rank Fusion | Gratuit (calcul local) |
| `mmr` | Diversité des résultats | Gratuit (calcul local) |
| `cross_encoder` | Reranking via LLM | 1 appel LLM |
| `node_distance` | Proximité dans le graphe | Gratuit |
| `episode_mentions` | Fréquence de mention | Gratuit |

## Problème résolu : `/v1/responses`

graphiti-core v0.28.2 utilise l'**API Responses** d'OpenAI (`/v1/responses`) pour l'extraction structurée. LiteLLM ne supporte pas cet endpoint.

**Solution** : `_ProxyCompatibleOpenAIClient` dans `graphiti_client.py` override `_generate_response` pour toujours utiliser `/v1/chat/completions` avec JSON mode. Le schéma Pydantic est injecté dans le prompt au lieu d'utiliser le parsing natif.

## Erreurs rencontrées et solutions

### Sprint 1 (LiteLLM)

| # | Erreur | Cause | Solution |
|---|--------|-------|----------|
| 1 | `orpheam-network` not found | docker-compose prod dépend du réseau externe | Utiliser `docker-compose.test.yml` |
| 2 | Healthcheck `unhealthy` | `curl` absent dans l'image LiteLLM | Healthcheck via `python urllib` |
| 3 | Healthcheck 401 | `/health` requiert auth | Utiliser `/health/readiness` |
| 4 | `LLM Provider NOT provided` | litellm local essaie de résoudre le provider | `custom_llm_provider="openai"` |
| 5 | `Missing Anthropic API Key` | Fallbacks exécutés localement | Supprimer `fallbacks=` côté client |
| 6 | `No api key passed in` | Router n'envoyait pas la master key | Ajout param `api_key` |
| 7 | `Result.is_ok()` AttributeError | API Result utilise `.ok` (propriété) | Remplacer par `.ok` |
| 8 | Model IDs invalides | Noms incorrects sur OpenRouter | Corriger dans `litellm_config.yaml` |
| 9 | `completion_cost()` error | API changée | Utiliser `cost_per_token()` |
| 10 | Venv vide | Poetry cache vs .venv local | `poetry install`, `pip install -e` |

### Sprint 2 (Graphiti)

| # | Erreur | Cause | Solution |
|---|--------|-------|----------|
| 11 | `build_indices` Cypher error | Graphe existant + `delete_existing=True` | `delete_existing=False` |
| 12 | `add_episode` 400 error | graphiti utilise `/v1/responses` (pas supporté par LiteLLM) | `_ProxyCompatibleOpenAIClient` override |
| 13 | `neo4j_service.py` param mismatch | Ancien param `gemini_api_key` | Refactoré vers `api_key` + `base_url` |
| 14 | Notebook mauvais kernel | `.venv` parasite dans `ia/` | Supprimer, sélectionner `ia/workers/.venv` |

## Prochaines étapes

- [ ] Valider notebook 03 (add_episode + search)
- [ ] Tester `OrpheamSearchConfig` avec différentes recettes
- [ ] Publier orpheam-libs sur Nexus
- [ ] Config prod : Redis cache, budget/rate limits dans litellm_config.yaml
- [ ] Audit pipeline KG v3 vs Graphiti natif
