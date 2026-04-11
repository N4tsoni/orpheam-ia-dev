---
tags: [concept, kg]
aliases: [KG, Graphe de connaissances]
categorie: structure de donnees
---

# Knowledge Graph

## Definition

Structure de donnees representant des connaissances sous forme de triplets (sujet, predicat, objet). Les entites sont des noeuds, les relations sont des aretes typees et orientees.

## Fonctionnement

- **Entites** — objets du monde reel (personnes, organisations, concepts)
- **Relations** — liens types entre entites (travaille_pour, est_un, localise_a)
- **Proprietes** — attributs sur les noeuds et aretes (date, source, confiance)
- **Temporalite** — certains KG gerent la validite temporelle des faits (voir [[Temporal Knowledge Graph]])

## Cas d'usage

- GraphRAG — enrichir le contexte LLM avec des faits structures
- Deduplication d'entites multi-sources
- Raisonnement multi-hop (traversee de relations)
- Orpheam : pipeline Promethee (documents -> KG -> RAG)

## Implementations

- [[Neo4j GDS]] — base graphe native + algorithmes
- [[promethee/Graphiti]] — KG temporel avec extraction LLM
- Wikidata, DBpedia — KG publics

## Concepts lies

- [[Temporal Knowledge Graph]]
- [[Named Entity Recognition]]
- [[Community Detection]]
- [[Centrality]]
- [[Link Prediction]]
