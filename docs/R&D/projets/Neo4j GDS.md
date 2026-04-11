---
tags: [projet, gds]
url: "https://neo4j.com/docs/graph-data-science/"
licence: "Community + Enterprise"
langage: "Java"
maintenu: true
---

# Neo4j GDS

> **URL** : https://neo4j.com/docs/graph-data-science/
> **Langage** : Java (plugin Neo4j)
> **Licence** : Community (algos de base) + Enterprise (algos avances)

## Description

Librairie d'algorithmes de graphe integree a Neo4j. Fonctionne sur des projections in-memory du graphe pour des performances optimales. 60+ algorithmes couvrant centralite, communautes, similarite, path finding, embeddings.

## Architecture

1. **Graph Projection** — projection du graphe Neo4j en memoire (native ou Cypher)
2. **Algorithm Execution** — execution sur la projection (stream, stats, mutate, write)
3. **Result** — retour des resultats (stream vers client ou ecriture dans Neo4j)

Modes d'execution :
- `stream` — resultats en flux (pas de persistence)
- `mutate` — ecrit dans la projection in-memory
- `write` — ecrit dans Neo4j
- `stats` — statistiques aggregees uniquement

## Integration Orpheam

- Sprint 3 prevu : enrichir le KG Graphiti avec des metriques GDS
- [[Centrality]] (PageRank) pour ponderer les entites dans le RAG
- [[Community Detection]] (Louvain/Leiden) pour segmenter le graphe
- Node similarity pour ameliorer la deduplication d'entites

## Concepts lies

- [[Centrality]]
- [[Community Detection]]
- [[Link Prediction]]
- [[Knowledge Graph]]

## Papers associes

- [[]]
