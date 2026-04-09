# Promethee — Pipeline Knowledge Graph

**Prométhée**, titan grec qui descend dans le monde des dieux pour ramener le feu sacré aux hommes. Ici, les données désorganisées sont les enfers — un chaos de documents bruts, de formats hétérogenes, d'informations enfouies. Prométhée plonge dans ce chaos et en ramene une ame : un graphe de connaissances structuré, vivant et exploitable.

---

Promethee transforme des **documents bruts** (CSV, PDF, DOCX, JSON, XLSX, TXT) en un **graphe de connaissances temporel** dans Neo4j. C'est le moteur qui alimente le KG qu'Apollon interroge.

## Flux simplifié

```
Document → Parsing → Extraction LLM → Graphe Neo4j
```

1. Un fichier arrive via RabbitMQ (job depuis Laravel)
2. Le **parser** détecte le format et découpe en morceaux (épisodes)
3. **Graphiti** ingere chaque épisode : le LLM extrait les entités et relations, déduplique, résout les entités similaires
4. Le graphe Neo4j est mis a jour — jamais de suppression, les faits obsoletes recoivent une date d'expiration
5. Optionnel : **entity linking** enrichit les entités avec Wikidata/DBpedia
6. Le résultat est renvoyé a Laravel

## Ce que Graphiti gere vs code custom

| | Graphiti natif | Notre code |
|---|:---:|:---:|
| Extraction entités/relations (LLM) | x | |
| Entity resolution (dedup fuzzy) | x | |
| Temporalité (valid_at / expired_at) | x | |
| Communities & sagas | x | |
| Embeddings | x | |
| Storage Neo4j | x | |
| Parsing fichiers multi-format | | x |
| Orchestration LangGraph | | x |
| Entity linking externe | | x |

> Graphiti couvre ~85% du pipeline nativement. Notre code custom gere le parsing, l'orchestration et l'enrichissement externe.

## Docs du dossier

- [Stack technique](Stack%20technique.md) — Technos, modeles LLM, taches Celery, fichiers clés
- [Graphiti](Graphiti.md) — Knowledge graph temporel (extraction, dedup, recherche)
- [Neo4j & Cypher](Neo4j%20&%20Cypher.md) — Base graphe, schéma, requetes utiles
- [Entity Linking](Entity%20Linking.md) — Enrichissement Wikidata / DBpedia
- [Evaluation Pipeline KG](Evaluation%20Pipeline%20KG%20-%20Guide.md) — Métriques, gold standard, benchmarks
- [Recap - Integration LiteLLM & Graphiti](Recap%20-%20Integration%20LiteLLM%20&%20Graphiti.md) — Journal technique Sprint 1+2
