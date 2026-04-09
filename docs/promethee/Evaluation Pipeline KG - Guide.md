# Évaluation du Pipeline KG — Guide Pratique

Comment mesurer la qualité de notre pipeline et comparer des configs.

---

## 1. Vue d'ensemble

Le pipeline a 3 dimensions à évaluer :

| Dimension | Question | Métriques |
|-----------|----------|-----------|
| **Extraction** | Est-ce que les bonnes entités/relations sont extraites ? | Precision, Recall, F1 |
| **Retrieval** | Est-ce que la recherche retourne le bon contexte ? | Context Precision/Recall |
| **Coût** | Combien ça coûte en tokens, temps, argent ? | Tokens, latence, $/fichier |

---

## 2. Évaluation de l'extraction

### 2.1 Gold Standard (méthode manuelle)

La base : annoter manuellement 30-50 chunks et comparer avec la sortie du pipeline.

**Étapes :**
1. Prendre 10-20 fichiers de test variés (CSV, PDF, DOCX...)
2. Pour chaque chunk, lister manuellement les entités et relations attendues
3. Lancer le pipeline → récupérer les entités/relations extraites par Graphiti
4. Comparer : Precision (pas de faux positifs), Recall (pas d'oublis), F1

**Format gold standard** (JSON) :
```json
{
  "chunk_id": "test_data_rows_1_5",
  "source_text": "Alice Martin est CTO de Orpheam...",
  "expected_entities": [
    {"name": "Alice Martin", "type": "Person"},
    {"name": "Orpheam", "type": "Organization"}
  ],
  "expected_relations": [
    {"source": "Alice Martin", "target": "Orpheam", "type": "CTO_OF"}
  ]
}
```

**Calcul :**
- Entity Match : exact + fuzzy (Levenshtein > 0.85 = match)
- Relation Match : triplet (source, relation, target) — les 3 doivent matcher
- Precision = vrais positifs / (vrais positifs + faux positifs)
- Recall = vrais positifs / (vrais positifs + faux négatifs)
- F1 = 2 × P × R / (P + R)

### 2.2 LLM-as-Judge (méthode scalable)

Pour évaluer à plus grande échelle sans tout annoter à la main.

**Principe :** Donner au LLM le texte source + les entités extraites, demander un score.

```python
prompt = f"""Évalue la qualité de l'extraction d'entités à partir de ce texte.

Texte source : {chunk_text}

Entités extraites : {entities}

Score de 1 à 5 :
- 1 : extraction très incomplète ou incorrecte
- 3 : extraction correcte mais manque des éléments importants
- 5 : extraction complète et précise

Réponds en JSON : {{"score": N, "missing": [...], "incorrect": [...]}}
"""
```

### 2.3 Requêtes de diagnostic Neo4j

Après un run, vérifier la santé du graphe :

```cypher
-- Distribution des types d'entités
MATCH (n:Entity) RETURN n.entity_type, count(n) ORDER BY count(n) DESC

-- Noeuds orphelins (pas de relations = extraction pauvre)
MATCH (n:Entity) WHERE NOT (n)-[:RELATES_TO]-() RETURN count(n)

-- Densité du graphe (ratio edges/nodes, idéalement > 1.5)
MATCH (n:Entity) WITH count(n) AS nodes
MATCH ()-[r:RELATES_TO]->() WITH nodes, count(r) AS edges
RETURN edges, nodes, toFloat(edges) / nodes AS density

-- Doublons potentiels (noms similaires)
MATCH (a:Entity), (b:Entity)
WHERE a <> b AND apoc.text.levenshteinSimilarity(a.name, b.name) > 0.85
RETURN a.name, b.name LIMIT 20

-- Distribution des degrés (détecter les hubs aberrants)
MATCH (n:Entity)
WITH n, size([(n)-[]-() | 1]) AS degree
RETURN n.name, degree ORDER BY degree DESC LIMIT 15
```

---

## 3. Évaluation du retrieval (recherche KG)

### 3.1 Dataset de test

Créer 30-50 questions avec réponses attendues :

```json
{
  "question": "Qui est la CTO de Orpheam ?",
  "expected_answer": "Alice Martin",
  "expected_entities": ["Alice Martin", "Orpheam"],
  "expected_facts": ["Alice Martin est CTO de Orpheam"]
}
```

### 3.2 Métriques retrieval

| Métrique | Mesure | Bon score |
|----------|--------|-----------|
| **Context Recall** | Le contexte retourné contient-il les faits nécessaires ? | > 0.8 |
| **Context Precision** | Le contexte retourné est-il pertinent (pas de bruit) ? | > 0.7 |
| **Hit Rate** | La bonne entité apparaît dans les top-K résultats ? | > 0.9 |
| **MRR** (Mean Reciprocal Rank) | Position moyenne du premier bon résultat | > 0.7 |

### 3.3 Framework RAGAS

[ragas.io](https://docs.ragas.io) — fonctionne avec LangChain/LangGraph.

```python
from ragas import evaluate
from ragas.metrics import context_precision, context_recall, faithfulness

# Pour chaque question du dataset :
# 1. Lancer graphiti.search(question)
# 2. Collecter le contexte retourné (edges.fact, nodes.summary)
# 3. Évaluer avec RAGAS
```

Alternative : **DeepEval** (`deepeval`) — plus simple, custom metrics faciles.

---

## 4. Évaluation coût / performance

### 4.1 Métriques à tracker par run

| Métrique | Source | Comment |
|----------|--------|---------|
| `tokens_in` | Callbacks LiteLLM | Total tokens envoyés |
| `tokens_out` | Callbacks LiteLLM | Total tokens reçus |
| `cost_usd` | OpenRouter billing | Coût réel du run |
| `latency_ms` | `stages[].duration_seconds` | Temps par étape |
| `entities_count` | `entities_extracted` | Output du pipeline |
| `relations_count` | `relations_extracted` | Output du pipeline |
| `entities_per_dollar` | entities / cost | Efficacité |

### 4.2 Où tracker

Le `KGPipelineState` a déjà `stages` avec les durées. Pour les tokens, LiteLLM expose des callbacks :

```python
import litellm
litellm.success_callback = ["custom_callback"]
# Ou via le proxy : GET /spend/logs
```

---

## 5. Benchmarks A/B avec KGPipelineConfig

### 5.1 Principe

Fixer le dataset, varier UNE dimension à la fois :

```python
from orpheam.graph.pipeline import KGPipelineConfig

config_a = KGPipelineConfig(
    name="gemini-flash",
    llm_model="google/gemini-2.5-flash",
    csv_batch_size=20,
)

config_b = KGPipelineConfig(
    name="claude-sonnet",
    llm_model="anthropic/claude-sonnet-4",
    csv_batch_size=20,
)

# Voir les différences
print(config_a.diff(config_b))
# → {'name': ('gemini-flash', 'claude-sonnet'), 'llm_model': ('google/gemini-2.5-flash', 'anthropic/claude-sonnet-4')}
```

### 5.2 Variables à tester

| Variable | Options à comparer |
|----------|-------------------|
| **Modèle LLM** | Gemini Flash vs Claude Sonnet vs GPT-4o |
| **Batch size CSV** | 5 vs 10 vs 20 vs 50 lignes |
| **Chunk size TXT** | 20 vs 50 vs 100 lignes |
| **Search reranker** | RRF vs MMR vs Cross-Encoder |
| **Search methods** | BM25 seul vs Cosine seul vs BM25+Cosine |

### 5.3 Structure des résultats

Stocker les résultats dans un DataFrame pour comparer :

```python
import pandas as pd

results = []
for config in [config_a, config_b]:
    pipeline = config.build_pipeline()
    # ... run pipeline, collect metrics ...
    results.append({
        "config": config.name,
        "llm_model": config.llm_model,
        "entities": state["entities_extracted"],
        "relations": state["relations_extracted"],
        "duration_s": sum(s["duration_seconds"] for s in state["stages"]),
        # "f1_entities": ...,  # si gold standard
        # "cost_usd": ...,    # si tracking activé
    })

df = pd.DataFrame(results)
print(df.to_markdown())
```

---

## 6. Checklist avant optimisation

Avant de modifier le pipeline, s'assurer d'avoir :

- [ ] Un jeu de données de test fixe (10-20 fichiers variés)
- [ ] Un gold standard pour au moins 30 chunks (entités + relations attendues)
- [ ] Un dataset de questions retrieval (30-50 questions)
- [ ] Les métriques de base avec la config par défaut (baseline)
- [ ] Un notebook d'évaluation reproductible

**Règle : pas d'optimisation sans mesure.** Chaque changement doit montrer une amélioration mesurable sur au moins une métrique sans dégrader les autres.

---

## 7. Priorité des évaluations

| Priorité | Évaluation | Effort | Impact |
|----------|-----------|--------|--------|
| 🔴 P0 | Diagnostic Neo4j (requêtes Cypher) | 30 min | Vérifie que le pipeline fonctionne |
| 🟠 P1 | Gold standard extraction (30 chunks) | 2-3h | Baseline F1 entités/relations |
| 🟡 P2 | Benchmark A/B modèles LLM | 1h | Choix du meilleur modèle qualité/coût |
| 🟢 P3 | Dataset retrieval + RAGAS | 2-3h | Qualité de la recherche |
| 🔵 P4 | Tracking coûts par run | 1h | Optimisation budget |
