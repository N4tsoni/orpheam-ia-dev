# Apollon — Agent Conversationnel

**Apollon**, dieu grec de la musique et orchestrateur des Muses. Il dirige, coordonne et fait jouer chaque instrument au bon moment. C'est exactement ce que fait notre agent : il orchestre les différents noeuds (intent, recherche KG, NER, LLM...) pour composer une réponse harmonieuse a chaque question.

---

Apollon est l'**assistant IA** d'Orpheam. Il recoit une question, comprend l'intention, cherche dans le graphe de connaissances si c'est pertinent, et génere une réponse contextualisée.

## Flux simplifié

```
Question → Comprendre l'intention → Chercher dans le KG → Répondre
```

## 3 modes de fonctionnement

L'agent s'adapte selon la complexité configurée :

### Linear (défaut) — 6 noeuds actifs

Le plus simple. Toutes les questions passent par le KG.

```
NER → Retrieval → Ranking → Context → LLM → Memory → END
```

### Basic Orchestrator — 7 noeuds actifs

Ajoute un classifieur d'intention pour éviter de chercher dans le KG quand c'est inutile.

```
Intent Classifier →
    kg_query → NER → Retrieval → Ranking → Context → LLM → Memory → END
    general  → Context → LLM → Memory → END
    direct   → LLM → Memory → END
```

### Intelligent Orchestrator — 10 noeuds actifs

Le plus avancé. Analyse la pertinence du KG, découpe les questions complexes, choisit la meilleure stratégie.

```
Intent → KG Awareness → Query Decomposition → Intelligent Router →
    FULL_KG  → NER → Retrieval → Ranking → Aggregator → LLM → Memory → END
    NO_MATCH → Aggregator → LLM → Memory → END
    DIRECT   → LLM → Memory → END
```

## Les 11 noeuds LangGraph

Tous sont définis dans le code, mais seuls certains sont actifs selon le mode :

| Noeud | Rôle | Linear | Basic | Intelligent |
|-------|------|:------:|:-----:|:-----------:|
| `intent_classifier` | Classifie la question (kg_query / general / direct) | | x | x |
| `ner_extraction` | Extrait les entités nommées | x | x | x |
| `semantic_retrieval` | Recherche vectorielle dans Neo4j | x | x | x |
| `ranking` | Score et classe les résultats KG | x | x | x |
| `context_building` | Formate le contexte pour le LLM | x | x | |
| `llm_call` | Appelle le LLM (OrpheamRouter) | x | x | x |
| `memory_persist` | Sauvegarde l'historique (Redis) | x | x | x |
| `kg_awareness` | Evalue si le KG est pertinent (score 0-1) | | | x |
| `query_decomposition` | Découpe en sous-taches | | | x |
| `intelligent_router` | Choisit la route (FULL_KG / NO_MATCH / DIRECT) | | | x |
| `result_aggregator` | Combine les résultats (remplace context_building) | | | x |

> En mode Intelligent, `context_building` est remplacé par `result_aggregator` qui gere aussi l'agrégation de sous-taches.

## Docs du dossier

- [Stack technique](Stack%20technique.md) — Technos, tache Celery, fichiers clés
- [LangGraph](LangGraph.md) — Machine a états, principe, state
- [Modes](Modes.md) — Détail des 3 modes de fonctionnement
