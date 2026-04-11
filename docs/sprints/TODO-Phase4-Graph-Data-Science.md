# TODO Phase 4 — Graph Data Science

> Objectif : exploiter la science des graphes pour maximiser la valeur des KG construits par Promethee.
>
> Détails techniques dans `docs/Plan.md`.

---

## 4.1 Neo4j GDS — Algorithmes classiques
- [ ] **Centralité** — PageRank, Betweenness, Degree, Eigenvector sur les KG Orpheam
- [ ] **Communautés** — Louvain, Leiden, Label Propagation
- [ ] **Similarité** — Node Similarity, Jaccard, Cosine structurelle
- [ ] **Chemins** — Shortest Path, All Shortest Paths, Dijkstra pondéré
- [ ] **Link Prediction classique** — Common Neighbors, Adamic-Adar, Preferential Attachment
- [ ] Multi-Factor Ranking (centralité + similarité sémantique + temporalité)
- [ ] Notebook de démonstration GDS
- [ ] Intégration dans les Muses comme source de contexte enrichi

## 4.2 Multi-Layer Graphs — Structures avancées
> Un graphe multi-couches sépare les relations par type/domaine sur des layers distincts,
> permettant des analyses croisées impossibles sur un graphe plat.

- [ ] **Architecture multi-layer** — design du schéma Neo4j (labels = layers, projections GDS par layer)
- [ ] **Kubernetes orchestration** — un namespace/deployment par layer de graph
  - Config registry : mapping `graph_id` → connexion Neo4j (host, port, credentials)
  - Routing Celery : queue par layer, workers dédiés par graph
  - Auto-scaling : HPA basé sur la taille du graph et la charge
- [ ] **Types de layers** à tester :
  - Layer sémantique (entités + relations extraites par LLM)
  - Layer structurel (hiérarchie documents, sections, paragraphes)
  - Layer temporel (évolution des entités dans le temps, Graphiti)
  - Layer externe (Wikidata, DBpedia, liens enrichis)
- [ ] **Cross-layer queries** — requêtes traversant plusieurs layers
  - Projections multi-graphes GDS
  - Fusion de résultats inter-layers (union, intersection, pondération)
- [ ] **Multi-DB Neo4j** — une base par domaine/client (isolation multi-tenant)
  - Neo4j Fabric pour requêtes fédérées cross-DB
  - Ou multiple instances Neo4j derrière un routeur
- [ ] Notebook multi-layer : créer 2-3 layers, exécuter des queries croisées
- [ ] Benchmark : graph plat vs multi-layer (qualité retrieval, latence)

## 4.3 LightRAG — RAG léger complémentaire
> Approche RAG légère et performante, complémentaire au pipeline KG.

- [ ] Benchmark LightRAG vs pipeline actuel (qualité retrieval, latence, coût)
- [ ] POC notebook avec dataset existant
- [ ] Décision : complémentaire (en parallèle de Graphiti) ou remplacement
- [ ] Intégration comme source de contexte alternative dans les Muses
- [ ] Évaluation RAGAS comparée au KG Graphiti
