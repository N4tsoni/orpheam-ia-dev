---
tags: [concept, gds, community-detection]
aliases: [Detection de communautes, Clustering de graphe]
categorie: algorithme GDS
---

# Community Detection

## Definition

Identification de groupes de noeuds densement connectes entre eux et faiblement connectes aux autres groupes. Revele la structure modulaire d'un graphe.

## Fonctionnement

Principaux algorithmes :
- **Louvain** — maximisation de modularite, hierarchique, rapide (O(n log n))
- **Leiden** — amelioration de Louvain, garantit des communautes connexes
- **Label Propagation** — iteratif, tres rapide, non-deterministe
- **Weakly Connected Components** — composantes connexes basiques

## Cas d'usage

- Segmentation thematique d'un [[Knowledge Graph]]
- Regroupement d'entites similaires pour deduplication
- Identification de clusters de documents lies
- Sprint 3 Orpheam : analyse structurelle du KG apres ingestion

## Implementations

- [[Neo4j GDS]] — `gds.louvain`, `gds.leiden`, `gds.labelPropagation`
- NetworkX — `community.louvain_communities()`
- igraph — `community_leiden()`

## Concepts lies

- [[Centrality]]
- [[Graph Neural Network]]
- [[Link Prediction]]
