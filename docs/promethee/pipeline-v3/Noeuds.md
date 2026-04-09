# Pipeline V3 — Les 7 Noeuds LangGraph

## Vue d'ensemble

```
CLASSIQUE:       [1] Entity → [2] Relation → [3] Validation → [4] Storage
NEURO-SYMB:  [5] Markdown → [6] SCT → [1] Entity → [2] Relation → [7] ASP → [3] Validation → [4] Storage
```

---

## 1. EntityExtractionNode

**Rôle** : Extraire les entités du texte via LLM avec JSON mode forcé.

**Etapes** :
1. Construit le prompt (few-shot + conditions SCT si dispo + contexte graphe)
2. Appelle le LLM avec `response_format={"type": "json_object"}` → 100% JSON valide
3. Parse la réponse avec `JSONResponseParser` (robuste aux erreurs courantes)
4. Valide chaque entité via Pydantic (`EntityOutput`)

**Sortie** : liste d'entités avec `type`, `name`, `properties`, `confidence`, `extraction_method`

Types reconnus : Person, Organization, Location, Technology, Project, Concept, Event, Document, Product

---

## 2. RelationExtractionNode

**Rôle** : Extraire les relations entre entités via **deux passes** LLM.

**Pass 1 — Relations explicites** : directement énoncées dans le texte
**Pass 2 — Relations implicites** : inférées du contexte (mode conservateur, haute confiance uniquement)

Les conditions SCT sont injectées dans les prompts (filtrées pour les entités déja extraites). Chaque relation est validée : `from_entity` et `to_entity` doivent matcher une entité existante (case-insensitive).

**Sortie** : liste de relations avec `type`, `from_entity`, `to_entity`, `confidence`, `evidence[]`

Types : WORKS_AT, MANAGES, LOCATED_IN, USES, DEVELOPS, COLLABORATES_WITH...

---

## 3. ValidationNode

**Rôle** : Déduplication et cohérence (pas d'appel LLM, logique pure Python).

1. **Dédup entités** par clé `(type, name)` case-insensitive
2. **Validation relations** : source et cible doivent exister dans les entités validées
3. **Normalisation** : confiance ramenée a [0.0, 1.0]

---

## 4. StorageNode

**Rôle** : Stockage dans Neo4j.

1. Convertit les dicts en objets enrichis (ajout source fichier, confidence, method)
2. `create_entities_batch()` puis `create_relations_batch()`
3. Met a jour les métriques finales

---

## 5. MarkdownParsingNode (neuro-symbolique)

**Rôle** : Parser le Markdown structuré en chunks exploitables.

Types de chunks reconnus :

| Type | Détection |
|------|-----------|
| `heading` | `## Titre` (H1-H6) — construit une hiérarchie `section_path` |
| `table` | `\|...\|` — parsé en colonnes + lignes (list of dicts) |
| `code_block` | ` ``` ... ``` ` — avec langue détectée |
| `list` | `- item` ou `1. item` — items groupés |
| `image_ref` | `![caption](ref)` — caption + référence |
| `paragraph` | Tout le reste |

Les chunks sont ensuite sérialisés en `raw_data` compatible avec les noeuds d'extraction.

**Pass-through** : si pas de Markdown → ce noeud ne fait rien, le chemin classique s'active.

---

## 6. SCTNode — Semantic Condition Transformer (neuro-symbolique)

**Rôle** : Enrichir les prompts LLM avec du contexte graphe calculé par un GNN.

C'est le coeur de l'approche neuro-symbolique. Il connecte le **graphe existant** (Neo4j) aux **prompts LLM** pour améliorer l'extraction.

**Pipeline** :
1. **Extraction candidats** — regex sur les noms propres (mots capitalisés)
2. **Voisinage Neo4j** — requete Cypher batch pour récupérer les voisins (profondeur 2)
3. **GNN Forward** — embedding SentenceTransformer + aggregation des voisins (1 couche)
4. **Adaptive Fusion** — score de cohérence + formatage textuel

**Résultat** : pour chaque entité candidate, un bloc de contexte :
```
[ Alice Johnson ]
  → Type(s) voisin(s) : Person (5), Organization (3)
  → Voisinage KG (7 noeuds, profondeur 2) :
    - TechCorp --[WORKS_AT]--> Alice Johnson
    - Paris --[LOCATED_IN]--> TechCorp
  → Entités liées : TechCorp, Paris, Berlin
  → Cohérence : 0.78 (élevée — entité bien connue)
```

Ce contexte est injecté **avant les données** dans les prompts EntityExtraction et RelationExtraction.

Voir [Neuro-Symbolique](Neuro-Symbolique.md) pour le détail du GNN et de l'Adaptive Fusion.

---

## 7. ASPVerificationNode (neuro-symbolique)

**Rôle** : Vérification logique des extractions via Answer Set Programming (clingo).

Apres l'extraction LLM, ce noeud traduit les entités/relations en **faits logiques**, applique des **regles de contrainte**, détecte les **violations**, et **répare automatiquement**.

**Pipeline** :
1. **Traduction** — entités/relations → faits ASP (`entity("alice", "person").`)
2. **Résolution** — clingo charge 3 fichiers de regles et cherche les violations
3. **Réparation** — chaque violation est corrigée selon sa stratégie

**Violations détectées** :

| Violation | Exemple | Réparation |
|-----------|---------|-----------|
| `reflexive_relation` | Alice → Alice | Supprime la relation |
| `dangling_source` | Relation sans source connue | Supprime la relation |
| `type_conflict` | Alice = Person ET Organization | Garde celle a confiance max |
| `schema_mismatch` | WORKS_AT: Org → Org (devrait etre Person → Org) | Downgrade confiance -0.2 |
| `confidence_out_of_range` | confiance > 1.0 | Normalise a [0.0, 1.0] |

Voir [Neuro-Symbolique](Neuro-Symbolique.md) pour le détail des regles ASP.
