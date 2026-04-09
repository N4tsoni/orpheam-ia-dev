# Graphiti — Knowledge Graph Temporel

## C'est quoi ?

[Graphiti](https://github.com/getzep/graphiti) (par Zep) est un moteur qui transforme du **texte brut en graphe de connaissances** dans Neo4j. Il gère automatiquement :

- **Extraction** — entités et relations via LLM
- **Entity Resolution** — dedup fuzzy (deux mentions de la meme entité = un seul noeud)
- **Temporalité** — les faits ont une date de validité, jamais de suppression
- **Communities** — clustering automatique d'entités liées
- **Embeddings** — vectorisation pour la recherche sémantique

## Ingestion (`add_episode`)

```
Texte brut + reference_time
    → LLM : extraction d'entités et relations
    → Dedup contre le graphe existant
    → Gestion temporelle (expired_at sur les faits obsolètes)
    → Embedding des entités et relations
    → Persistance Neo4j
```

Un "épisode" = un morceau de texte ingéré. Graphiti garde la trace de chaque épisode et des entités qu'il a générées (relation `MENTIONS`).

## Recherche (`search`)

```
Query en langage naturel
    → Embedding de la query
    → Recherche parallèle :
        1. Vector search (cosine similarity)
        2. Full-text search (BM25)
        3. BFS optionnel (parcours du graphe)
    → Reranking
    → Résultats triés
```

### Modes de recherche (OrpheamSearchConfig)

| Recette | Description | Coût |
|---------|-------------|------|
| `hybrid_rrf()` | BM25 + cosine, reranking rapide | Gratuit |
| `hybrid_cross_encoder()` | BM25 + cosine + BFS, reranking LLM | 1 appel LLM |
| `hybrid_mmr()` | Favorise la diversité des résultats | Gratuit |
| `edges_only()` | Faits/relations uniquement | - |
| `nodes_only()` | Entités uniquement | - |

```python
from orpheam.graph import OrpheamSearchConfig

config = OrpheamSearchConfig.hybrid_rrf()
results = await graphiti.search("query", config=config)
```

## Temporalité

Graphiti ne supprime **jamais** de données :
- Quand un fait change → l'ancien recoit `expired_at`, le nouveau a `valid_at`
- On peut requeter l'état du graphe à n'importe quelle date

## Notre intégration

Graphiti attend l'API OpenAI (`/v1/responses`), mais on passe par LiteLLM qui ne supporte pas cet endpoint.

**Solution** : `_ProxyCompatibleOpenAIClient` dans `graphiti_client.py` — override pour utiliser `/v1/chat/completions` + JSON mode a la place.

## Ce que Graphiti gère vs code custom

| Fonctionnalité | Graphiti natif | Code custom |
|----------------|:-:|:-:|
| Extraction entités/relations | x | |
| Entity resolution (dedup) | x | |
| Temporalité | x | |
| Communities/sagas | x | |
| Embedding | x | |
| Storage Neo4j | x | |
| Parsing fichiers | | x |
| Orchestration LangGraph | | x |
| Entity linking externe | | x |
