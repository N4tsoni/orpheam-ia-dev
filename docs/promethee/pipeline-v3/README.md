# Pipeline V3 — Extraction KG Neuro-Symbolique

> Développé par **Lazare (Laz)** — intégré dans la nouvelle archi le 9 avril 2026.

Le Pipeline V3.1 est un systeme d'extraction de graphes de connaissances qui combine **LLM** (extraction flexible) et **IA symbolique** (contraintes logiques). Il offre deux chemins d'exécution automatiquement sélectionnés selon le type d'entrée.

## Deux chemins

### Chemin Classique (CSV/JSON) — 4 noeuds

```
EntityExtraction → RelationExtraction → Validation → Storage
```

Le LLM extrait entités et relations, on valide, on stocke dans Neo4j. Simple et efficace.

### Chemin Neuro-Symbolique (Markdown) — 7 noeuds

```
MarkdownParsing → SCT → EntityExtraction → RelationExtraction → ASPVerification → Validation → Storage
```

En plus du chemin classique, on ajoute :
- **SCT** : un mini-GNN qui analyse le voisinage Neo4j des entités candidates et injecte du contexte dans les prompts LLM
- **ASP Verification** : un solveur logique (clingo) qui vérifie la cohérence des extractions et répare automatiquement les violations

Le choix est **automatique** : si `source_markdown` est fourni → chemin neuro-symbolique, sinon → classique.

## Performances

| Métrique | Avant | Pipeline V3 |
|----------|-------|-------------|
| Parsing JSON | ~60% | **100%** (forced JSON mode) |
| Temps (10 records) | ~54s | **~46s** (-15%) |

## Dégradation gracieuse

Tout est concu pour tourner meme si des composants manquent :

| Composant absent | Comportement |
|------------------|-------------|
| LangGraph | Exécution séquentielle (meme logique) |
| SentenceTransformer | GNN avec embeddings hash-based |
| Neo4j | SCT retourne des conditions vides |
| clingo | ASP bypassed, pass-through |

## Docs du dossier

- [Noeuds](Noeuds.md) — Les 7 noeuds LangGraph en détail
- [Neuro-Symbolique](Neuro-Symbolique.md) — SCT, GNN, ASP, Adaptive Fusion
- [Prompts & Parsers](Prompts%20&%20Parsers.md) — Construction des prompts LLM et parsing des réponses
