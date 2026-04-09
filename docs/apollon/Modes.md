# Apollon — Modes de fonctionnement

3 modes activables par configuration, du plus simple au plus avancé.

---

## Mode Linear (défaut)

Pipeline simple sans routing.

```
NER → Retrieval → Ranking → Context → LLM → Memory → END
```

Toutes les questions passent par le KG. Simple mais pas optimal si la question n'a rien a voir avec le graphe.

---

## Mode Basic Orchestrator (Sprint 12)

Ajoute une classification d'intent pour router.

```
Intent Classifier →
    kg_query → NER → Retrieval → Ranking → Context → LLM → Memory → END
    general  → LLM → Memory → END
    direct   → Memory → END
```

Evite de chercher dans le KG quand c'est pas nécessaire.

---

## Mode Intelligent Orchestrator (Sprint 13)

Routing avancé avec décomposition de query et analyse de pertinence KG.

```
Intent → KG Awareness → Query Decomposition → Intelligent Router →
    FULL_KG  → NER → Retrieval → Ranking → Context → LLM → Aggregator → Memory → END
    HYBRID   → [KG + LLM en parallele] → Aggregator → Memory → END
    DIRECT   → LLM → Memory → END
    NO_MATCH → LLM (sans KG) → Memory → END
```

**KG Awareness** : regarde si le graphe contient des données pertinentes (score 0-1).
**Query Decomposition** : découpe les questions complexes en sous-tâches.
**Intelligent Router** : choisit la meilleure stratégie selon le score KG + l'intent.
