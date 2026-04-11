---
tags: [projet, kg, temporal]
url: "https://github.com/getzep/graphiti"
licence: "Apache 2.0"
langage: "Python"
maintenu: true
---

# Graphiti

> **URL** : https://github.com/getzep/graphiti
> **Langage** : Python (graphiti-core)
> **Licence** : Apache 2.0

## Description

Framework de [[Temporal Knowledge Graph]] par Zep. Extrait automatiquement entites et relations depuis du texte via LLM, gere la deduplication, la temporalite (faits expires vs actifs), et persiste dans Neo4j.

## Architecture

- **add_episode()** — ingestion : texte -> extraction LLM -> dedup -> Neo4j
- **search()** — recherche hybride (fulltext + vector + reranker)
- **Temporalite** — chaque fait a `created_at` / `expired_at`, jamais supprime
- **Chunking interne** — gere le decoupage du texte en interne

## Integration Orpheam

- Coeur du pipeline Promethee (Sprint 2)
- Wrapper : `OrpheamGraphitiClient` dans orpheam-libs
- LLM via proxy LiteLLM (OpenAI-compatible)
- Pipeline : parse-worker -> Markdown -> Graphiti -> Neo4j
- Version actuelle : graphiti-core 0.3.21

## Concepts lies

- [[Temporal Knowledge Graph]]
- [[Knowledge Graph]]
- [[Named Entity Recognition]]

## Papers associes

- [[]]
