# Recherche — Graph Data Science

Notes de lecture et veille sur les knowledge graphs, graph data science, et techniques associees.

## Organisation

```
recherche/
  papers/       — Fiches de lecture par article (1 note = 1 paper)
  concepts/     — Concepts cles (algorithmes, techniques, metriques)
  auteurs/      — Chercheurs et collegues (affiliations, travaux)
  projets/      — Projets / outils open-source (Neo4j GDS, Graphiti, LightRAG...)
```

## Conventions

### Tags

Utiliser des tags normalises pour faciliter l'extraction KG :

| Tag | Usage |
|-----|-------|
| `#paper` | Fiche de lecture |
| `#concept` | Definition d'un concept |
| `#auteur` | Fiche chercheur |
| `#projet` | Outil / framework |
| `#gds` | Graph Data Science |
| `#kg` | Knowledge Graphs |
| `#ner` | Named Entity Recognition |
| `#embedding` | Graph / text embeddings |
| `#evaluation` | Metriques, benchmarks |
| `#temporal` | Graphes temporels |
| `#community-detection` | Detection de communautes |
| `#centrality` | Algorithmes de centralite |
| `#link-prediction` | Prediction de liens |
| `#gnn` | Graph Neural Networks |

### Liens

- Lier les papers aux concepts : `[[PageRank]]`, `[[Community Detection]]`
- Lier les papers aux auteurs : `[[Nom Prenom]]`
- Lier les concepts entre eux quand il y a une relation directe
- Utiliser des alias si necessaire : `[[Graph Neural Networks|GNN]]`

### Vocabulaire normalise

Pour eviter les doublons d'entites dans le KG, utiliser systematiquement :

| Forme canonique | Pas |
|-----------------|-----|
| Knowledge Graph | KG, knowledge graph, graphe de connaissances |
| Graph Data Science | GDS, graph data science |
| Neo4j GDS | Neo4j Graph Data Science Library |
| Named Entity Recognition | NER |
| Graph Neural Network | GNN |
| Community Detection | detection de communautes |
| Link Prediction | prediction de liens |
| Temporal Knowledge Graph | TKG, graphe temporel |

## Workflow

1. Importer le PDF dans **Zotero** (collection "Graph Data Science")
2. Lire et annoter dans Zotero
3. Creer une fiche dans `papers/` a partir du template
4. Lier aux `concepts/` et `auteurs/` existants (ou les creer)
5. Taguer correctement
6. Optionnel : alimenter le pipeline KG avec la fiche pour test/benchmark
