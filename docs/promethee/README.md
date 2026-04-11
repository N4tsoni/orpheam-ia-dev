# Promethee — Pipeline Knowledge Graph

**Prométhée**, titan grec qui descend dans le monde des dieux pour ramener le feu sacré aux hommes. Ici, les données désorganisées sont les enfers — un chaos de documents bruts, de formats hétérogènes, d'informations enfouies. Prométhée plonge dans ce chaos et en ramène une âme : un graphe de connaissances structuré, vivant et exploitable.

---

Promethee transforme des **documents bruts** (CSV, PDF, DOCX, JSON, XLSX, TXT) en un **graphe de connaissances temporel** dans Neo4j. C'est le moteur qui alimente le KG qu'Apollon interroge.

## Flux simplifié

```
Document → Parsing → Extraction LLM → Graphe Neo4j
```

1. Un fichier arrive via RabbitMQ (job depuis Laravel)
2. Le **parser** détecte le format et découpe en morceaux (épisodes)
3. **Graphiti** ingère chaque épisode : le LLM extrait les entités et relations, déduplique, résout les entités similaires
4. Le graphe Neo4j est mis à jour — jamais de suppression, les faits obsolètes reçoivent une date d'expiration
5. Optionnel : **entity linking** enrichit les entités avec Wikidata/DBpedia
6. Le résultat est renvoyé à Laravel

## Ce que Graphiti gère vs code custom

|                                     | Graphiti natif | Notre code |
| ----------------------------------- | :------------: | :--------: |
| Extraction entités/relations (LLM)  |       x        |            |
| Entity resolution (dedup fuzzy)     |       x        |            |
| Temporalité (valid_at / expired_at) |       x        |            |
| Communities & sagas                 |       x        |            |
| Embeddings                          |       x        |            |
| Storage Neo4j                       |       x        |            |
| Parsing fichiers multi-format       |                |     x      |
| Orchestration LangGraph             |                |     x      |
| Entity linking externe              |                |     x      |

> Graphiti couvre ~85% du pipeline nativement. Notre code custom gère le parsing, l'orchestration et l'enrichissement externe.

## Sous-dossiers

- [graphe/](graphe/) — Graphiti, Neo4j & Cypher, Entity Linking
- [parsing/](parsing/) — Parser Worker, options d'intégration MinIO/Laravel
- [pipeline-v3/](pipeline-v3/) — Pipeline neuro-symbolique (par Laz) : nœuds, ASP, GNN
- [evaluation/](evaluation/) — Métriques, gold standard, benchmarks
