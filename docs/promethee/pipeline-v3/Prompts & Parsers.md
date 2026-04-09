# Pipeline V3 — Prompts & Parsers

## Prompts LLM

### EntityPromptBuilder

Construit le prompt d'extraction d'entités en 5 blocs :

1. **Few-shot examples** (optionnel) — 2 exemples annotés
2. **Conditions SCT** (si dispo) — contexte graphe injecté avant les données (max 10)
3. **Graph context** (optionnel) — entités existantes dans Neo4j
4. **Instruction** : "Extract ALL entities: people, organizations, locations, technologies..."
5. **Données** : records JSON a traiter

Le system prompt impose :
- Output = JSON array strict (pas de texte avant/apres)
- Chaque entité doit avoir : `type`, `name`, `properties`, `confidence`, `extraction_method`
- Methods : `explicit` (dit dans le texte), `inferred` (déduit), `context` (via contexte graphe)

### RelationPromptBuilder

Similaire mais pour les relations. Deux modes :
- `explicit` : relations directement énoncées
- `implicit` : relations inférées (mode conservateur)

Les conditions SCT sont **filtrées** pour n'inclure que les entités déja extraites. Le prompt inclut la liste des entités disponibles pour que le LLM ne référence que des entités existantes.

---

## Parsers

### JSONResponseParser — Robustesse maximale

Les LLM ne retournent pas toujours du JSON propre. Ce parser gère :

1. Supprime les code blocks markdown (` ```json ... ``` `)
2. Extrait le JSON depuis le premier `{` ou `[`
3. Corrige les trailing commas (`, }` → `}`)
4. Gère le JSON imbriqué (`{"items": [...]}` → extrait la liste)

### EntityParser / RelationParser — Validation Pydantic

```python
class EntityOutput(BaseModel):
    type: str                    # Person, Organization, Location...
    name: str                    # non-vide
    properties: dict = {}
    confidence: float            # [0.0, 1.0]
    extraction_method: str       # explicit | inferred | context | manual

class RelationOutput(BaseModel):
    type: str                    # WORKS_AT, MANAGES...
    from_entity: str
    to_entity: str
    properties: dict = {}
    confidence: float            # [0.0, 1.0]
    extraction_method: str       # explicit | inferred | context
    evidence: list[str] = []     # textes justificatifs
```

### MarkdownParser — Chunks structurés

Parse le Markdown brut en `MarkdownChunk[]` avec :
- Hiérarchie des headings (`section_path`)
- Tables → list of dicts
- Code blocks avec langue
- Listes groupées
- Références d'images

### CacheManager — Cache sémantique

Cache intelligent basé sur la similarité des embeddings :
- Seuil : 0.95 (95% de similarité → cache hit)
- TTL : 24h
- LRU eviction (max 1000 items)
- Utilise SentenceTransformer pour comparer les requetes
