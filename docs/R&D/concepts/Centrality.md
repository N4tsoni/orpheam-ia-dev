---
tags: [concept, gds, centrality]
aliases: [Centralite, Mesures de centralite]
categorie: algorithme GDS
---

# Centrality

## Definition

Mesures quantifiant l'importance relative d'un noeud dans un graphe. Permet d'identifier les entites cles, les hubs d'information, les points de passage critiques.

## Fonctionnement

Principaux algorithmes :
- **PageRank** — importance basee sur les liens entrants ponderes recursivement
- **Betweenness Centrality** — noeuds sur le plus grand nombre de chemins courts
- **Degree Centrality** — nombre de connexions directes (in/out)
- **Closeness Centrality** — proximite moyenne a tous les autres noeuds
- **Eigenvector Centrality** — importance basee sur l'importance des voisins

## Cas d'usage

- Identifier les entites les plus importantes du [[Knowledge Graph]]
- Ponderer les resultats de recherche dans le RAG
- Detecter les concepts-pivot entre domaines
- Sprint 3 Orpheam : enrichir le ranking des resultats Graphiti

## Implementations

- [[Neo4j GDS]] — `gds.pageRank`, `gds.betweenness`, `gds.degree`
- NetworkX — `nx.pagerank()`, `nx.betweenness_centrality()`

## Concepts lies

- [[Community Detection]]
- [[Link Prediction]]
- [[Knowledge Graph]]
